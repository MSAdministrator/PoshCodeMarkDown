---
Author: tome tanasovski
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1695
Published Date: 
Archived Date: 2010-03-31t09
---

# set-winschedule - 

## Description

set-winschedule gives a gui to select a schedule and schedules a task using schtasks.  this is a beta.  there are still a lot of features to implement.  please read through the synopsis->description to see the list of features that i hope to get in a final release.

## Comments



## Usage



## TODO



## script

`set-winschedule`

## Code

`#
 #
 
 [reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 [reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
 
 function Set-WinSchedule {
     <#
       .Synopsis
         Creates a winform to select a schedule and creates a scheduled task
        .Description
         Set-WinSchedule gives a GUI to select a schedule and schedules a task using schtasks
         This is a beta.  There are still a lot of features to implement:
             Need to have more scheduling options.  I expect to have all options available in a recurring outlook calendar item
             Need to have methods for scheduling with all 3 providers: schtasks, wmi, and at.  Currently it only uses schtasks
             Need to design the return object properties:
                 Should contain the text paths for each provider type
                 Should contain a date/time for start time
             Need to provide a method to overwrite an existing task if it has the same name and the user confirms that it is ok to overwrite.  Should also provide
             a -force parameter for this option.
             Need to ensure that files piped from get-item will be scheduled
             Need a parameter to override ok box at the end
        .Example
         Set-WinSchedule c:\windows\notepad.exe notepadtask
        .Parameter TaskRun
         The name of the command to be scheduled
        .Parameter ScheduleName
         The name that the scheduled task will be given.   
        .Notes
         NAME:  Set-Schedule
         AUTHOR: Tome Tanasovski
         LASTEDIT: 3/11/2010
         KEYWORDS:
        .Link
          http://powertoe.wordpress.com
      #>
     param(
         [Parameter(Position=1,Mandatory=$true)]
         [string] $taskrun,
         [Parameter(Position=2,Mandatory=$true)]
         [string] $taskname
     )
     $command = "& schtasks.exe /query /tn $taskname"
     $job = start-job $ExecutionContext.InvokeCommand.NewScriptBlock($command)
     Wait-Job $job
     if ($job.ChildJobs[0].output -ne "") {  
         [System.windows.forms.messagebox]::show("A task named $taskname already exists.  You must delete this task before you can use the name.")
         return        
     }
     
     
     $SchedulePickerForm = New-Object System.Windows.Forms.Form
     $comboTime = New-Object System.Windows.Forms.ComboBox
     $label4 = New-Object System.Windows.Forms.Label
     $buttonCancel = New-Object System.Windows.Forms.Button
     $buttonOK = New-Object System.Windows.Forms.Button
     $group = New-Object System.Windows.Forms.GroupBox
     $checkSaturday = New-Object System.Windows.Forms.CheckBox
     $checkFriday = New-Object System.Windows.Forms.CheckBox
     $checkThursday = New-Object System.Windows.Forms.CheckBox
     $checkWednesday = New-Object System.Windows.Forms.CheckBox
     $checkTuesday = New-Object System.Windows.Forms.CheckBox
     $checkMonday = New-Object System.Windows.Forms.CheckBox
     $checkSunday = New-Object System.Windows.Forms.CheckBox
     $labelDays = New-Object System.Windows.Forms.Label
     $labelHours = New-Object System.Windows.Forms.Label
     $boxHourlyDaily = New-Object System.Windows.Forms.TextBox
     $labelEvery = New-Object System.Windows.Forms.Label
     $radioHourly = New-Object System.Windows.Forms.RadioButton
     $radioWeekly = New-Object System.Windows.Forms.RadioButton
     $radioDaily = New-Object System.Windows.Forms.RadioButton
     $InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
     $checkboxes = ($checkMonday,$checkTuesday,$checkWednesday,$checkThursday,$checkFriday,$checkSaturday,$checkSunday)
 
     function VisibleInvisibleCheckBoxes {
         Write-Host $checkboxes
         $checkboxes |foreach {$_.visible = -not $_.visible}
     }
     $handler_radioButtonChanged = {    
         switch ($true) {
             ($radioHourly.Checked) {
                 $labelHours.Visible = $true
                 $labelDays.Visible = $false
                 $boxHourlyDaily.Visible = $true
                 $checkboxes |foreach {$_.visible = $false}
             }
             ($radioDaily.Checked) {
                 $labelHours.Visible = $false
                 $labelDays.Visible = $true
                 $boxHourlyDaily.Visible = $true
                 $checkboxes |foreach {$_.visible = $false}
             }
             ($radioWeekly.Checked) {
                 $labelHours.Visible = $false
                 $labelDays.Visible = $false
                 $boxHourlyDaily.Visible = $false
                 $checkboxes |foreach {$_.visible = $true}
             }
         }
     }
 
     $buttonCancel_OnClick = {
         $SchedulePickerForm.Close()
         return $null
     }
 
     $buttonOK_OnClick = {
         $doit = $false
         switch ($true) {
             ($radioHourly.Checked -or $radioDaily.Checked) {
                 try {
                     $recurrence = [Convert]::ToInt32($boxHourlyDaily.Text)
                     if ($recurrence -gt 0) {
                         try {
                             switch ($true) {
                                 ($radiohourly.checked) {
                                     if ($recurence -gt 23) {
                                         [System.windows.forms.messagebox]::show("Hourly recurrence must be 1-23 hours")
                                         $boxHourlyDaily.Focus()
                                     }
                                     else {                                
                                         & schtasks /create /tn $taskname /tr "$taskrun" /sc hourly /mo $boxHourlyDaily.Text /st $comboTime.Text /f
                                         [System.Windows.Forms.Messagebox]::show("Task has been scheduled")
                                         $SchedulePickerForm.Close()                                                                        
                                     }
                                 }
                                 ($radioDaily.checked) {
                                     if ($recurence -gt 365) {
                                         [System.windows.forms.messagebox]::show("Hourly recurrence must be 1-365 hours")
                                         $boxhourlydaily.focus()
                                     }
                                     else {
                                         & schtasks /create /tn $taskname /tr $taskrun /sc daily /mo $boxHourlyDaily.Text /st $comboTime.Text /f
                                         $SchedulePickerForm.Close()                                
                                     }
                                 }
                             }                            
                         }
                         catch {
                             [System.windows.forms.messagebox]::show($error[0])
                         }
                     }
                     else {
                         [System.windows.forms.messagebox]::show("Recurrence must be greater than 0")
                         $boxHourlyDaily.Focus()
                     }
                 }
                 catch {
                     [System.windows.forms.messagebox]::show("You must enter a valid integer recurrence")
                     $boxHourlyDaily.Focus()
                 }
             }
             ($radioWeekly.Checked) {
                 $dflag = ""
                 $checkboxes|foreach {
                     if ($_.checked) {
                         $dflag += $_.text.substring(0,3) + ","                        
                     }
                 }
                 if ($dflag -ne "") {        
                     $dflag = $dflag.substring(0,$dflag.length-1)
                     & schtasks /create /tn $taskname /tr $taskrun /sc weekly /st $comboTime.Text /d "$dflag" /f
                     $SchedulePickerForm.Close()
                 }
                 else {
                     [System.windows.forms.messagebox]::show("You must select at least one day for weekly recurrence")
                 }
             }
         }
 
     }
 
     $OnLoadForm_StateCorrection={
     	$SchedulePickerForm.WindowState = $InitialFormWindowState
     }
 
     $SchedulePickerForm.Text = "Schedule Picker"
     $SchedulePickerForm.MaximizeBox = $False
     $SchedulePickerForm.Name = "SchedulePickerForm"
     $SchedulePickerForm.DataBindings.DefaultDataSourceUpdateMode = 0
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 476
     $System_Drawing_Size.Height = 157
     $SchedulePickerForm.ClientSize = $System_Drawing_Size
     $SchedulePickerForm.FormBorderStyle = 5
 
     $comboTime.FormattingEnabled = $True
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 121
     $System_Drawing_Size.Height = 21
     $comboTime.Size = $System_Drawing_Size
     $comboTime.DataBindings.DefaultDataSourceUpdateMode = 0
     $comboTime.Name = "comboTime"
     $comboTime.Items.Add("00:00")|Out-Null
     $comboTime.Items.Add("00:30")|Out-Null
     $comboTime.Items.Add("01:00")|Out-Null
     $comboTime.Items.Add("01:30")|Out-Null
     $comboTime.Items.Add("02:00")|Out-Null
     $comboTime.Items.Add("02:30")|Out-Null
     $comboTime.Items.Add("03:00")|Out-Null
     $comboTime.Items.Add("03:30")|Out-Null
     $comboTime.Items.Add("04:00")|Out-Null
     $comboTime.Items.Add("04:30")|Out-Null
     $comboTime.Items.Add("05:00")|Out-Null
     $comboTime.Items.Add("05:30")|Out-Null
     $comboTime.Items.Add("06:00")|Out-Null
     $comboTime.Items.Add("06:30")|Out-Null
     $comboTime.Items.Add("07:00")|Out-Null
     $comboTime.Items.Add("07:30")|Out-Null
     $comboTime.Items.Add("08:00")|Out-Null
     $comboTime.Items.Add("08:30")|Out-Null
     $comboTime.Items.Add("09:00")|Out-Null
     $comboTime.Items.Add("09:30")|Out-Null
     $comboTime.Items.Add("10:00")|Out-Null
     $comboTime.Items.Add("10:30")|Out-Null
     $comboTime.Items.Add("11:00")|Out-Null
     $comboTime.Items.Add("11:30")|Out-Null
     $comboTime.Items.Add("12:00")|Out-Null
     $comboTime.Items.Add("12:30")|Out-Null
     $comboTime.Items.Add("13:00")|Out-Null
     $comboTime.Items.Add("13:30")|Out-Null
     $comboTime.Items.Add("14:00")|Out-Null
     $comboTime.Items.Add("14:30")|Out-Null
     $comboTime.Items.Add("15:00")|Out-Null
     $comboTime.Items.Add("15:30")|Out-Null
     $comboTime.Items.Add("16:00")|Out-Null
     $comboTime.Items.Add("16:30")|Out-Null
     $comboTime.Items.Add("17:00")|Out-Null
     $comboTime.Items.Add("17:30")|Out-Null
     $comboTime.Items.Add("18:00")|Out-Null
     $comboTime.Items.Add("18:30")|Out-Null
     $comboTime.Items.Add("19:00")|Out-Null
     $comboTime.Items.Add("19:30")|Out-Null
     $comboTime.Items.Add("20:00")|Out-Null
     $comboTime.Items.Add("20:30")|Out-Null
     $comboTime.Items.Add("21:00")|Out-Null
     $comboTime.Items.Add("21:30")|Out-Null
     $comboTime.Items.Add("22:00")|Out-Null
     $comboTime.Items.Add("22:30")|Out-Null
     $comboTime.Items.Add("23:00")|Out-Null
     $comboTime.Items.Add("23:30")|Out-Null
     $comboTime.Text = "08:00"
     $comboTime.DropDownStyle = "DropDownList"
     
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 53
     $System_Drawing_Point.Y = 119
     $comboTime.Location = $System_Drawing_Point
     $comboTime.TabIndex = 1
 
     $SchedulePickerForm.Controls.Add($comboTime)
 
     $label4.TabIndex = 3
     $label4.TextAlign = 16
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 35
     $System_Drawing_Size.Height = 23
     $label4.Size = $System_Drawing_Size
     $label4.Text = "Start:"
 
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 12
     $System_Drawing_Point.Y = 116
     $label4.Location = $System_Drawing_Point
     $label4.DataBindings.DefaultDataSourceUpdateMode = 0
     $label4.Name = "label4"
 
     $SchedulePickerForm.Controls.Add($label4)
 
     $buttonCancel.TabIndex = 3
     $buttonCancel.Name = "buttonCancel"
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 75
     $System_Drawing_Size.Height = 23
     $buttonCancel.Size = $System_Drawing_Size
     $buttonCancel.UseVisualStyleBackColor = $True
 
     $buttonCancel.Text = "Cancel"
 
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 368
     $System_Drawing_Point.Y = 119
     $buttonCancel.Location = $System_Drawing_Point
     $buttonCancel.DataBindings.DefaultDataSourceUpdateMode = 0
     $buttonCancel.add_Click($buttonCancel_OnClick)
 
     $SchedulePickerForm.Controls.Add($buttonCancel)
 
     $buttonOK.TabIndex = 2
     $buttonOK.Name = "buttonOK"
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 75
     $System_Drawing_Size.Height = 23
     $buttonOK.Size = $System_Drawing_Size
     $buttonOK.UseVisualStyleBackColor = $True
 
     $buttonOK.Text = "OK"
 
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 287
     $System_Drawing_Point.Y = 119
     $buttonOK.Location = $System_Drawing_Point
     $buttonOK.DataBindings.DefaultDataSourceUpdateMode = 0
     $buttonOK.add_Click($buttonOK_OnClick)
 
     $SchedulePickerForm.Controls.Add($buttonOK)
 
     $group.Name = "group"
 
     $group.Text = "Recurrence pattern"
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 431
     $System_Drawing_Size.Height = 101
     $group.Size = $System_Drawing_Size
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 12
     $System_Drawing_Point.Y = 12
     $group.Location = $System_Drawing_Point
     $group.TabStop = $False
     $group.TabIndex = 0
     $group.DataBindings.DefaultDataSourceUpdateMode = 0
 
     $SchedulePickerForm.Controls.Add($group)
 
     $checkSaturday.UseVisualStyleBackColor = $True
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 73
     $System_Drawing_Size.Height = 24
     $checkSaturday.Size = $System_Drawing_Size
     $checkSaturday.TabIndex = 13
     $checkSaturday.Text = "Saturday"
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 274
     $System_Drawing_Point.Y = 64
     $checkSaturday.Location = $System_Drawing_Point
     $checkSaturday.DataBindings.DefaultDataSourceUpdateMode = 0
     $checkSaturday.Name = "checkSaturday"
 
     $checkSaturday.Visible = $False
 
     $group.Controls.Add($checkSaturday)
 
 
     $checkFriday.UseVisualStyleBackColor = $True
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 64
     $System_Drawing_Size.Height = 24
     $checkFriday.Size = $System_Drawing_Size
     $checkFriday.TabIndex = 12
     $checkFriday.Text = "Friday"
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 204
     $System_Drawing_Point.Y = 64
     $checkFriday.Location = $System_Drawing_Point
     $checkFriday.DataBindings.DefaultDataSourceUpdateMode = 0
     $checkFriday.Name = "checkFriday"
 
     $checkFriday.Visible = $False
 
     $group.Controls.Add($checkFriday)
 
 
     $checkThursday.UseVisualStyleBackColor = $True
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 70
     $System_Drawing_Size.Height = 24
     $checkThursday.Size = $System_Drawing_Size
     $checkThursday.TabIndex = 11
     $checkThursday.Text = "Thursday"
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 129
     $System_Drawing_Point.Y = 64
     $checkThursday.Location = $System_Drawing_Point
     $checkThursday.DataBindings.DefaultDataSourceUpdateMode = 0
     $checkThursday.Name = "checkThursday"
 
     $checkThursday.Visible = $False
 
     $group.Controls.Add($checkThursday)
 
 
     $checkWednesday.UseVisualStyleBackColor = $True
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 83
     $System_Drawing_Size.Height = 24
     $checkWednesday.Size = $System_Drawing_Size
     $checkWednesday.TabIndex = 10
     $checkWednesday.Text = "Wednesday"
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 342
     $System_Drawing_Point.Y = 44
     $checkWednesday.Location = $System_Drawing_Point
     $checkWednesday.DataBindings.DefaultDataSourceUpdateMode = 0
     $checkWednesday.Name = "checkWednesday"
 
     $checkWednesday.Visible = $False
 
     $group.Controls.Add($checkWednesday)
 
 
     $checkTuesday.UseVisualStyleBackColor = $True
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 68
     $System_Drawing_Size.Height = 24
     $checkTuesday.Size = $System_Drawing_Size
     $checkTuesday.TabIndex = 9
     $checkTuesday.Text = "Tuesday"
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 274
     $System_Drawing_Point.Y = 44
     $checkTuesday.Location = $System_Drawing_Point
     $checkTuesday.DataBindings.DefaultDataSourceUpdateMode = 0
     $checkTuesday.Name = "checkTuesday"
 
     $checkTuesday.Visible = $False
 
     $group.Controls.Add($checkTuesday)
 
 
     $checkMonday.UseVisualStyleBackColor = $True
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 64
     $System_Drawing_Size.Height = 24
     $checkMonday.Size = $System_Drawing_Size
     $checkMonday.TabIndex = 8
     $checkMonday.Text = "Monday"    
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 204
     $System_Drawing_Point.Y = 44
     $checkMonday.Location = $System_Drawing_Point
     $checkMonday.DataBindings.DefaultDataSourceUpdateMode = 0
     $checkMonday.Name = "checkMonday"
 
     $checkMonday.Visible = $False
 
     $group.Controls.Add($checkMonday)
 
 
     $checkSunday.UseVisualStyleBackColor = $True
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 104
     $System_Drawing_Size.Height = 24
     $checkSunday.Size = $System_Drawing_Size
     $checkSunday.TabIndex = 7
     $checkSunday.Text = "Sunday"
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 129
     $System_Drawing_Point.Y = 44
     $checkSunday.Location = $System_Drawing_Point
     $checkSunday.DataBindings.DefaultDataSourceUpdateMode = 0
     $checkSunday.Name = "checkSunday"
 
     $checkSunday.Visible = $False
 
     $group.Controls.Add($checkSunday)
 
     $labelDays.TabIndex = 6
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 64
     $System_Drawing_Size.Height = 18
     $labelDays.Size = $System_Drawing_Size
     $labelDays.Visible = $False
     $labelDays.Text = "day(s)"
 
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 189
     $System_Drawing_Point.Y = 23
     $labelDays.Location = $System_Drawing_Point
     $labelDays.DataBindings.DefaultDataSourceUpdateMode = 0
     $labelDays.Name = "labelDays"
 
     $group.Controls.Add($labelDays)
 
     $labelHours.TabIndex = 5
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 100
     $System_Drawing_Size.Height = 23
     $labelHours.Size = $System_Drawing_Size
     $labelHours.Text = "hour(s)"
 
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 189
     $System_Drawing_Point.Y = 23
     $labelHours.Location = $System_Drawing_Point
     $labelHours.DataBindings.DefaultDataSourceUpdateMode = 0
     $labelHours.Name = "labelHours"
 
     $group.Controls.Add($labelHours)
 
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 28
     $System_Drawing_Size.Height = 20
     $boxHourlyDaily.Size = $System_Drawing_Size
     $boxHourlyDaily.DataBindings.DefaultDataSourceUpdateMode = 0
     $boxHourlyDaily.Text = "1"
     $boxHourlyDaily.Name = "boxHourlyDaily"
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 155
     $System_Drawing_Point.Y = 20
     $boxHourlyDaily.Location = $System_Drawing_Point
     $boxHourlyDaily.TabIndex = 4
 
     $group.Controls.Add($boxHourlyDaily)
 
     $labelEvery.TabIndex = 3
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 67
     $System_Drawing_Size.Height = 23
     $labelEvery.Size = $System_Drawing_Size
     $labelEvery.Text = "Every"
 
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 116
     $System_Drawing_Point.Y = 23
     $labelEvery.Location = $System_Drawing_Point
     $labelEvery.DataBindings.DefaultDataSourceUpdateMode = 0
     $labelEvery.Name = "labelEvery"
 
     $group.Controls.Add($labelEvery)
 
     $radioHourly.TabIndex = 0
     $radioHourly.Name = "radioHourly"
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 104
     $System_Drawing_Size.Height = 24
     $radioHourly.Size = $System_Drawing_Size
     $radioHourly.UseVisualStyleBackColor = $True
 
     $radioHourly.Text = "Hourly"
     $radioHourly.Checked = $True
 
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 6
     $System_Drawing_Point.Y = 17
     $radioHourly.Location = $System_Drawing_Point
     $radioHourly.DataBindings.DefaultDataSourceUpdateMode = 0
     $radioHourly.TabStop = $True
     $radioHourly.add_Click($handler_radioButtonChanged)
     
     $group.Controls.Add($radioHourly)
 
     $radioWeekly.TabIndex = 2
     $radioWeekly.Name = "radioWeekly"
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 104
     $System_Drawing_Size.Height = 24
     $radioWeekly.Size = $System_Drawing_Size
     $radioWeekly.UseVisualStyleBackColor = $True
 
     $radioWeekly.Text = "Weekly"
 
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 6
     $System_Drawing_Point.Y = 56
     $radioWeekly.Location = $System_Drawing_Point
     $radioWeekly.DataBindings.DefaultDataSourceUpdateMode = 0
     $radioWeekly.add_Click($handler_radioButtonChanged)
 
     $group.Controls.Add($radioWeekly)
 
     $radioDaily.TabIndex = 1
     $radioDaily.Name = "radioDaily"
     $System_Drawing_Size = New-Object System.Drawing.Size
     $System_Drawing_Size.Width = 104
     $System_Drawing_Size.Height = 24
     $radioDaily.Size = $System_Drawing_Size
     $radioDaily.UseVisualStyleBackColor = $True
 
     $radioDaily.Text = "Daily"
 
     $System_Drawing_Point = New-Object System.Drawing.Point
     $System_Drawing_Point.X = 6
     $System_Drawing_Point.Y = 37
     $radioDaily.Location = $System_Drawing_Point
     $radioDaily.DataBindings.DefaultDataSourceUpdateMode = 0
     $radioDaily.add_Click($handler_radioButtonChanged)
 
     $group.Controls.Add($radioDaily)
     
     $SchedulePickerForm.CancelButton = $buttonCancel
     $SchedulePickerForm.AcceptButton = $buttonOK
 
     $InitialFormWindowState = $SchedulePickerForm.WindowState
     $SchedulePickerForm.add_Load($OnLoadForm_StateCorrection)
     $SchedulePickerForm.ShowDialog() |out-null
 
 }
`

