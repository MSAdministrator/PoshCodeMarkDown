---
Author: bryan jaudon
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3781
Published Date: 2012-11-23t14
Archived Date: 2012-11-30t08
---

# exchange autoreply admin - 

## Description

exchange automatic replies administrator – a gui front-end to the get-mailboxautoreplyconfiguration and set-mailboxautoreplyconfiguration cmdlet available in exchange 2010. this script mimics the gui available in outlook. perfect for helpdesk or sysadmins that hate the command-line, but want an easy way to set another users’ automatic replies (out of office) status. has not been tested with exchange 2013. works in powershell 2.0 and 3.0. it currently requires the management tools be installed on the same computer as the script.

## Comments



## Usage



## TODO



## script

`status-disabledautoreply`

## Code

`#
 #
 <#
 
     .NOTES
     Name     : Exchange Automatic Replies Administrator.ps1
     Author   : Bryan Jaudon <bryan.jaudon@gmail.com>
     Version  : 1.0
     Date     : 11/23/2012
 
     .SYNOPSIS
     Provides a graphical front-end to the Get-MailboxAutoReplyConfiguration and
     Set-MailboxAutoReplyConfiguration Exchange cmdlets. Requires Exchange management
     tools be installed on the same system as the script.
 
 
 #>
 
 $Script:ScriptTitle="Exchange Automatic Replies Administrator"
 
 [reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 [reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
 [System.Windows.Forms.Application]::EnableVisualStyles()
 
 function GenerateForm {
 ########################################################################
 ########################################################################
 
 
 $form1 = New-Object System.Windows.Forms.Form
 $labelConnectedFQDN = New-Object System.Windows.Forms.Label
 $labelConnected = New-Object System.Windows.Forms.Label
 $buttonOK = New-Object System.Windows.Forms.Button
 $buttonCancel = New-Object System.Windows.Forms.Button
 $labelAutoreply = New-Object System.Windows.Forms.Label
 $tabControl1 = New-Object System.Windows.Forms.TabControl
 $tabPage1 = New-Object System.Windows.Forms.TabPage
 $textInside = New-Object System.Windows.Forms.TextBox
 $tabPage2 = New-Object System.Windows.Forms.TabPage
 $radioAll = New-Object System.Windows.Forms.RadioButton
 $radioKnown = New-Object System.Windows.Forms.RadioButton
 $checkReplyOutside = New-Object System.Windows.Forms.CheckBox
 $textOutside = New-Object System.Windows.Forms.TextBox
 $dateEndTime = New-Object System.Windows.Forms.DateTimePicker
 $dateEndDate = New-Object System.Windows.Forms.DateTimePicker
 $labelEndTime = New-Object System.Windows.Forms.Label
 $dateStartTime = New-Object System.Windows.Forms.DateTimePicker
 $dateStartDate = New-Object System.Windows.Forms.DateTimePicker
 $labelStartTime = New-Object System.Windows.Forms.Label
 $checkOnlySend = New-Object System.Windows.Forms.CheckBox
 $radioSend = New-Object System.Windows.Forms.RadioButton
 $radioDoNotSend = New-Object System.Windows.Forms.RadioButton
 $comboIdentity = New-Object System.Windows.Forms.ComboBox
 $labelIdentity = New-Object System.Windows.Forms.Label
 $labelTitle = New-Object System.Windows.Forms.Label
 $pbIcon = New-Object System.Windows.Forms.PictureBox
 $InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
 
 #----------------------------------------------
 #----------------------------------------------
 
 $dateEndDate_ValueChanged= 
 {
 
     $buttonOK.Enabled=$true    
 
 }
 
 $dateStartTime_ValueChanged= 
 {
 
     $buttonOK.Enabled=$true
 
 }
 
 $textOutside_TextChanged= 
 {
 
     $buttonOK.Enabled=$true
 
 }
 
 $textInside_TextChanged= 
 {
 
     $buttonOK.Enabled=$true
 
 }
 
 $dateStartDate_ValueChanged= 
 {
 
     $buttonOK.Enabled=$true
 
 }
 
 $dateEndTime_ValueChanged= 
 {
 
     $buttonOK.Enabled=$true
 
 }
 
 $handler_radioAll_Click=
 {
 
     $buttonOK.Enabled=$true
 
 }
 
 $handler_radioKnown_Click=
 {
 
     $buttonOK.Enabled=$true
 
 }
 
 $handler_checkReplyOutside_Click= 
 {
     if ($checkReplyOutside.checked) {
         $radioKnown.Enabled=$true
         $radioAll.Enabled=$true
         $textOutside.Enabled=$true
         $tabPage2.Text = "Outside My Organization (On)"
     }
     else {
         $radioKnown.Enabled=$false
         $radioAll.Enabled=$false
         $textOutside.Enabled=$false
         $tabPage2.Text = "Outside My Organization (Off)"
     }
     $buttonOK.Enabled=$true
         
 }
 
 $buttonOK_OnClick= 
 {
 
     $commandHashTable=@{ "Identity"=$comboIdentity.SelectedItem }
 
         if ($radioDoNotSend.Checked) { $commandHashTable.Add("AutoReplyState","Disabled") }
         else {
             if ($checkOnlySend.Checked) { 
                 $commandHashTable.Add("AutoReplyState","Scheduled") 
                 $commandHashTable.Add("StartTime",$(Get-Date -Month $dateStartDate.value.Month -Day $dateStartDate.Value.Day -Year $dateStartDate.value.Year -Hour $dateStartTime.Value.Hour -Minute $dateStartTime.value.Minute -Second $dateStartTime.value.Second))
                 $commandHashTable.Add("EndTime",$(Get-Date -Month $dateEndDate.value.Month -Day $dateEndDate.Value.Day -Year $dateEndDate.value.Year -Hour $dateEndTime.Value.Hour -Minute $dateEndTime.value.Minute -Second $dateEndTime.value.Second))
                 if ($commandHashTable["EndTime"] -le $commandHashTable["StartTime"]) { ShowPrompt -Message "The end time cannot be less than or equal to start time." -icon Error -Buttons OK;return }
                 if ($commandHashTable["EndTime"] -le (Get-Date)) { ShowPrompt -Message "The end time that you entered occurs before the current time." -icon Error -Buttons OK;return }
                 }
             else { $commandHashTable.Add("AutoReplyState","Enabled") }
 
             $commandHashTable.Add("InternalMessage",$textInside.Text)
             if ($checkReplyOutside.checked) {
                 $commandHashTable.Add("ExternalMessage",$textOutside.Text)
                 if ($radioAll.Checked) { $commandHashTable.Add("ExternalAudience","All") }
                 elseif ($radioKnown.Checked) { $commandHashTable.Add("ExternalAudience","Known") }
             
             }
             else { $commandHashTable.Add("ExternalAudience","None") }
         }
  
     try { $status=Get-MailboxAutoReplyConfiguration -Identity $comboIdentity.SelectedItem }
     catch { ShowPrompt -Message $_ -Icon Error -Buttons OK;return }
 
     $result=ShowPrompt -Message "You have chosen to change the Automatic Reply configuration for`n'$($comboIdentity.SelectedItem)' from the state of '$($status.AutoReplyState)' to the state of '$($commandhashtable["AutoReplyState"])'.`n`nDo you wish to commit this change?" -Buttons "YesNo" -icon Warning
 
     if ($result -eq "Yes") {
 
         try { Set-MailboxAutoReplyConfiguration @commandHashTable -ErrorAction STOP }
         catch { ShowPrompt -Message $_ -Icon Error -Buttons OK;return }
         
         try { $status=Get-MailboxAutoReplyConfiguration -Identity $comboIdentity.SelectedItem }
         catch { ShowPrompt -Message $_ -Icon Error -Buttons OK;return }
         
         if (($status.AutoReplyState -eq $commandHashTable["AutoReplyState"]) -and ($status.IsValid -eq $True)) {
             ShowPrompt -Message "Automatic Reply configuration successful!" -Icon Information -Buttons OK;GetAutoReplyStatus -identity $comboIdentity.SelectedItem;$buttonOK.Enabled=$false;$form1.Refresh();return
         }
         else { ShowPrompt -Message "Automatic Reply configuration failed. `n`nCurrent Status is: $($status.AutoReplyState)" -Icon Error -Buttons OK;GetAutoReplyStatus -identity $comboIdentity.SelectedItem;$form1.Refresh();return }
 
     }
     else { ShowPrompt -Message "Automatic Reply configuration cancelled!" -Icon Error -Buttons OK;return }
 
 }
 
 $OnLoadForm_StateCorrection=
     $form1.WindowState = $InitialFormWindowState
     $form1.Update() 
     try { $comboIdentity.items.AddRange($(Get-CASMailbox | sort -Property Identity | Select -ExpandProperty Identity)) }
     catch { ShowPrompt -Message $_ -Buttons OK -Icon Error }
 
     if ($comboIdentity.items.count -lt 1) { ShowPrompt -Message "There were no Mailboxes found in the Exchange Organization.`nExiting Application." -Buttons OK -Icon Error;$form1.Close() }
 }
 
 $comboIdentity_SelectionChangeCommitted=
 {
     GetAutoReplyStatus -identity $comboIdentity.SelectedItem
 }
 
 $handler_radioSend_Click=
 {
     $scheduledstate=$checkOnlySend.Checked
     Status-EnabledAutoReply
     $checkOnlySend.Checked=$scheduledstate
 }
 
 $handler_radioDoNotSend_Click=
 {
     $scheduledstate=$checkOnlySend.Checked
     Status-DisabledAutoReply
     $checkOnlySend.Checked=$scheduledstate
 }
 
 $handler_checkOnlySend_Click=
 {
     switch ($checkOnlySend.Checked) {
         true { Status-ScheduledAutoReply }
         false { Status-EnabledAutoReply }
     }
 }
 
 #----------------------------------------------
 $imageList = new-Object System.Windows.Forms.ImageList 
 $System_Drawing_Size = New-Object System.Drawing.Size 
 $System_Drawing_Size.Width = 24 
 $System_Drawing_Size.Height = 24 
 $imageList.ImageSize = $System_Drawing_Size 
 $image1 = $Insideicon
 $image2 = $Outsideicon
  
 $imageList.Images.Add("Inside",$image1) 
 $imageList.Images.Add("Outside",$image2) 
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 557
 $System_Drawing_Size.Width = 538
 $form1.ClientSize = $System_Drawing_Size
 $form1.DataBindings.DefaultDataSourceUpdateMode = 0
 $form1.FormBorderStyle = 1
 $form1.MaximizeBox = $False
 $form1.MinimizeBox = $False
 $form1.Name = "form1"
 $form1.Text = "Exchange Automatic Replies Administrator"
 $form1.Icon=$icon
 
 $labelConnectedFQDN.AutoEllipsis = $True
 $labelConnectedFQDN.DataBindings.DefaultDataSourceUpdateMode = 0
 $labelConnectedFQDN.Font = New-Object System.Drawing.Font("Microsoft Sans Serif",8.25,1,3,1)
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 120
 $System_Drawing_Point.Y = 522
 $labelConnectedFQDN.Location = $System_Drawing_Point
 $labelConnectedFQDN.Name = "labelConnectedFQDN"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 238
 $labelConnectedFQDN.Size = $System_Drawing_Size
 $labelConnectedFQDN.TabIndex = 18
 $labelConnectedFQDN.Text = "$connectedFqdn"
 $labelConnectedFQDN.TextAlign = 16
 
 $form1.Controls.Add($labelConnectedFQDN)
 
 $labelConnected.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 522
 $labelConnected.Location = $System_Drawing_Point
 $labelConnected.Name = "labelConnected"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 101
 $labelConnected.Size = $System_Drawing_Size
 $labelConnected.TabIndex = 17
 $labelConnected.Text = "Connected Server:"
 $labelConnected.TextAlign = 16
 
 $form1.Controls.Add($labelConnected)
 
 
 $buttonOK.DataBindings.DefaultDataSourceUpdateMode = 0
 $buttonOK.Enabled = $False
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 371
 $System_Drawing_Point.Y = 522
 $buttonOK.Location = $System_Drawing_Point
 $buttonOK.Name = "buttonOK"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $buttonOK.Size = $System_Drawing_Size
 $buttonOK.TabIndex = 16
 $buttonOK.Text = "OK"
 $buttonOK.UseVisualStyleBackColor = $True
 $buttonOK.add_Click($buttonOK_OnClick)
 
 $form1.Controls.Add($buttonOK)
 
 
 $buttonCancel.DataBindings.DefaultDataSourceUpdateMode = 0
 $buttonCancel.DialogResult = 2
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 452
 $System_Drawing_Point.Y = 522
 $buttonCancel.Location = $System_Drawing_Point
 $buttonCancel.Name = "buttonCancel"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $buttonCancel.Size = $System_Drawing_Size
 $buttonCancel.TabIndex = 15
 $buttonCancel.Text = "Cancel"
 $buttonCancel.UseVisualStyleBackColor = $True
 $buttonCancel.add_Click($buttonCancel_OnClick)
 
 $form1.Controls.Add($buttonCancel)
 
 $labelAutoreply.DataBindings.DefaultDataSourceUpdateMode = 0
 $labelAutoreply.Enabled = $False
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 234
 $labelAutoreply.Location = $System_Drawing_Point
 $labelAutoreply.Name = "labelAutoreply"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 514
 $labelAutoreply.Size = $System_Drawing_Size
 $labelAutoreply.TabIndex = 14
 $labelAutoreply.Text = "Automatically reply once for each sender with the following messages:"
 $labelAutoreply.TextAlign = 16
 
 $form1.Controls.Add($labelAutoreply)
 
 $tabControl1.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 25
 $System_Drawing_Size.Width = 119
 $tabControl1.ItemSize = $System_Drawing_Size
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 260
 $tabControl1.Location = $System_Drawing_Point
 $tabControl1.Name = "tabControl1"
 $tabControl1.SelectedIndex = 0
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 256
 $System_Drawing_Size.Width = 514
 $tabControl1.Size = $System_Drawing_Size
 $tabControl1.TabIndex = 13
 $tabControl1.ImageList=$imageList
 
 $form1.Controls.Add($tabControl1)
 $tabPage1.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 4
 $System_Drawing_Point.Y = 22
 $tabPage1.Location = $System_Drawing_Point
 $tabPage1.Name = "tabPage1"
 $System_Windows_Forms_Padding = New-Object System.Windows.Forms.Padding
 $System_Windows_Forms_Padding.All = 3
 $System_Windows_Forms_Padding.Bottom = 3
 $System_Windows_Forms_Padding.Left = 3
 $System_Windows_Forms_Padding.Right = 3
 $System_Windows_Forms_Padding.Top = 3
 $tabPage1.Padding = $System_Windows_Forms_Padding
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 230
 $System_Drawing_Size.Width = 506
 $tabPage1.Size = $System_Drawing_Size
 $tabPage1.TabIndex = 0
 $tabPage1.Text = "Inside My Organization"
 $tabPage1.UseVisualStyleBackColor = $True
 $tabPage1.ImageIndex=0
 
 $tabControl1.Controls.Add($tabPage1)
 $textInside.DataBindings.DefaultDataSourceUpdateMode = 0
 $textInside.Enabled = $False
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 6
 $System_Drawing_Point.Y = 7
 $textInside.Location = $System_Drawing_Point
 $textInside.Multiline = $True
 $textInside.Name = "textInside"
 $textInside.ScrollBars = 3
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 210
 $System_Drawing_Size.Width = 494
 $textInside.Size = $System_Drawing_Size
 $textInside.TabIndex = 0
 $textInside.add_TextChanged($textInside_TextChanged)
 
 $tabPage1.Controls.Add($textInside)
 
 
 $tabPage2.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 4
 $System_Drawing_Point.Y = 22
 $tabPage2.Location = $System_Drawing_Point
 $tabPage2.Name = "tabPage2"
 $System_Windows_Forms_Padding = New-Object System.Windows.Forms.Padding
 $System_Windows_Forms_Padding.All = 3
 $System_Windows_Forms_Padding.Bottom = 3
 $System_Windows_Forms_Padding.Left = 3
 $System_Windows_Forms_Padding.Right = 3
 $System_Windows_Forms_Padding.Top = 3
 $tabPage2.Padding = $System_Windows_Forms_Padding
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 230
 $System_Drawing_Size.Width = 506
 $tabPage2.Size = $System_Drawing_Size
 $tabPage2.TabIndex = 1
 $tabPage2.Text = "Outside My Organization (Off)"
 $tabPage2.UseVisualStyleBackColor = $True
 $tabPage2.ImageIndex=1
 
 
 $tabControl1.Controls.Add($tabPage2)
 
 $radioAll.DataBindings.DefaultDataSourceUpdateMode = 0
 $radioAll.Enabled = $False
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 141
 $System_Drawing_Point.Y = 23
 $radioAll.Location = $System_Drawing_Point
 $radioAll.Name = "radioAll"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 24
 $System_Drawing_Size.Width = 190
 $radioAll.Size = $System_Drawing_Size
 $radioAll.TabIndex = 3
 $radioAll.TabStop = $True
 $radioAll.Text = "Anyone outside my organization"
 $radioAll.UseVisualStyleBackColor = $True
 $radioAll.Checked=$True
 $radioAll.add_Click($handler_radioAll_Click)
 
 $tabPage2.Controls.Add($radioAll)
 
 
 $radioKnown.DataBindings.DefaultDataSourceUpdateMode = 0
 $radioKnown.Enabled = $False
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 25
 $System_Drawing_Point.Y = 23
 $radioKnown.Location = $System_Drawing_Point
 $radioKnown.Name = "radioKnown"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 24
 $System_Drawing_Size.Width = 110
 $radioKnown.Size = $System_Drawing_Size
 $radioKnown.TabIndex = 2
 $radioKnown.TabStop = $True
 $radioKnown.Text = "My Contacts only"
 $radioKnown.UseVisualStyleBackColor = $True
 $radioKnown.add_Click($handler_radioKnown_Click)
 
 $tabPage2.Controls.Add($radioKnown)
 
 
 $checkReplyOutside.DataBindings.DefaultDataSourceUpdateMode = 0
 $checkReplyOutside.Enabled = $False
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 6
 $System_Drawing_Point.Y = 3
 $checkReplyOutside.Location = $System_Drawing_Point
 $checkReplyOutside.Name = "checkReplyOutside"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 24
 $System_Drawing_Size.Width = 253
 $checkReplyOutside.Size = $System_Drawing_Size
 $checkReplyOutside.TabIndex = 1
 $checkReplyOutside.Text = "Auto-Reply to people outside my organization"
 $checkReplyOutside.UseVisualStyleBackColor = $True
 $checkReplyOutside.add_Click($handler_checkReplyOutside_Click)
 
 $tabPage2.Controls.Add($checkReplyOutside)
 
 $textOutside.DataBindings.DefaultDataSourceUpdateMode = 0
 $textOutside.Enabled = $False
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 6
 $System_Drawing_Point.Y = 53
 $textOutside.Location = $System_Drawing_Point
 $textOutside.Multiline = $True
 $textOutside.Name = "textOutside"
 $textOutside.ScrollBars = 3
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 164
 $System_Drawing_Size.Width = 494
 $textOutside.Size = $System_Drawing_Size
 $textOutside.TabIndex = 0
 $textOutside.add_TextChanged($textOutside_TextChanged)
 
 $tabPage2.Controls.Add($textOutside)
 
 
 
 $dateEndTime.CustomFormat = "h:mm tt"
 $dateEndTime.DataBindings.DefaultDataSourceUpdateMode = 0
 $dateEndTime.Enabled = $False
 $dateEndTime.Format = 8
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 282
 $System_Drawing_Point.Y = 204
 $dateEndTime.Location = $System_Drawing_Point
 $dateEndTime.Name = "dateEndTime"
 $dateEndTime.ShowUpDown = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 76
 $dateEndTime.Size = $System_Drawing_Size
 $dateEndTime.TabIndex = 12
 $dateEndTime.add_ValueChanged($dateEndTime_ValueChanged)
 
 $form1.Controls.Add($dateEndTime)
 
 $dateEndDate.CustomFormat = "ddd MM/dd/yyyy"
 $dateEndDate.DataBindings.DefaultDataSourceUpdateMode = 0
 $dateEndDate.Enabled = $False
 $dateEndDate.Format = 8
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 152
 $System_Drawing_Point.Y = 204
 $dateEndDate.Location = $System_Drawing_Point
 $dateEndDate.Name = "dateEndDate"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 124
 $dateEndDate.Size = $System_Drawing_Size
 $dateEndDate.TabIndex = 11
 $dateEndDate.add_ValueChanged($dateEndDate_ValueChanged)
 
 $form1.Controls.Add($dateEndDate)
 
 $labelEndTime.DataBindings.DefaultDataSourceUpdateMode = 0
 $labelEndTime.Enabled = $False
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 82
 $System_Drawing_Point.Y = 202
 $labelEndTime.Location = $System_Drawing_Point
 $labelEndTime.Name = "labelEndTime"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 64
 $labelEndTime.Size = $System_Drawing_Size
 $labelEndTime.TabIndex = 10
 $labelEndTime.Text = "End time:"
 $labelEndTime.TextAlign = 16
 
 $form1.Controls.Add($labelEndTime)
 
 $dateStartTime.CustomFormat = "h:mm tt"
 $dateStartTime.DataBindings.DefaultDataSourceUpdateMode = 0
 $dateStartTime.Enabled = $False
 $dateStartTime.Format = 8
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 282
 $System_Drawing_Point.Y = 178
 $dateStartTime.Location = $System_Drawing_Point
 $dateStartTime.Name = "dateStartTime"
 $dateStartTime.ShowUpDown = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 77
 $dateStartTime.Size = $System_Drawing_Size
 $dateStartTime.TabIndex = 9
 $dateStartTime.add_ValueChanged($dateStartTime_ValueChanged)
 
 $form1.Controls.Add($dateStartTime)
 
 $dateStartDate.CustomFormat = "ddd MM/dd/yyyy"
 $dateStartDate.DataBindings.DefaultDataSourceUpdateMode = 0
 $dateStartDate.Enabled = $False
 $dateStartDate.Format = 8
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 152
 $System_Drawing_Point.Y = 178
 $dateStartDate.Location = $System_Drawing_Point
 $dateStartDate.Name = "dateStartDate"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 124
 $dateStartDate.Size = $System_Drawing_Size
 $dateStartDate.TabIndex = 8
 $dateStartDate.add_ValueChanged($dateStartDate_ValueChanged)
 
 $form1.Controls.Add($dateStartDate)
 
 $labelStartTime.DataBindings.DefaultDataSourceUpdateMode = 0
 $labelStartTime.Enabled = $False
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 82
 $System_Drawing_Point.Y = 175
 $labelStartTime.Location = $System_Drawing_Point
 $labelStartTime.Name = "labelStartTime"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 64
 $labelStartTime.Size = $System_Drawing_Size
 $labelStartTime.TabIndex = 7
 $labelStartTime.Text = "Start time:"
 $labelStartTime.TextAlign = 16
 
 $form1.Controls.Add($labelStartTime)
 
 
 $checkOnlySend.DataBindings.DefaultDataSourceUpdateMode = 0
 $checkOnlySend.Enabled = $False
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 52
 $System_Drawing_Point.Y = 148
 $checkOnlySend.Location = $System_Drawing_Point
 $checkOnlySend.Name = "checkOnlySend"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 24
 $System_Drawing_Size.Width = 190
 $checkOnlySend.Size = $System_Drawing_Size
 $checkOnlySend.TabIndex = 6
 $checkOnlySend.Text = "Only send during this time range:"
 $checkOnlySend.UseVisualStyleBackColor = $True
 $checkOnlySend.add_Click($handler_checkOnlySend_Click)
 
 $form1.Controls.Add($checkOnlySend)
 
 
 $radioSend.DataBindings.DefaultDataSourceUpdateMode = 0
 $radioSend.Enabled = $False
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 125
 $radioSend.Location = $System_Drawing_Point
 $radioSend.Name = "radioSend"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 24
 $System_Drawing_Size.Width = 182
 $radioSend.Size = $System_Drawing_Size
 $radioSend.TabIndex = 5
 $radioSend.Text = "Send automatic replies"
 $radioSend.UseVisualStyleBackColor = $True
 $radioSend.add_Click($handler_radioSend_Click)
 
 $form1.Controls.Add($radioSend)
 
 
 $radioDoNotSend.Checked = $True
 $radioDoNotSend.DataBindings.DefaultDataSourceUpdateMode = 0
 $radioDoNotSend.Enabled = $False
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 104
 $radioDoNotSend.Location = $System_Drawing_Point
 $radioDoNotSend.Name = "radioDoNotSend"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 24
 $System_Drawing_Size.Width = 182
 $radioDoNotSend.Size = $System_Drawing_Size
 $radioDoNotSend.TabIndex = 4
 $radioDoNotSend.TabStop = $True
 $radioDoNotSend.Text = "Do not send automatic replies"
 $radioDoNotSend.UseVisualStyleBackColor = $True
 $radioDoNotSend.add_Click($handler_radioDoNotSend_Click)
 
 $form1.Controls.Add($radioDoNotSend)
 
 $comboIdentity.DataBindings.DefaultDataSourceUpdateMode = 0
 $comboIdentity.DropDownStyle = 2
 $comboIdentity.FormattingEnabled = $True
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 69
 $System_Drawing_Point.Y = 67
 $comboIdentity.Location = $System_Drawing_Point
 $comboIdentity.Name = "comboIdentity"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 21
 $System_Drawing_Size.Width = 458
 $comboIdentity.Size = $System_Drawing_Size
 $comboIdentity.TabIndex = 3
 $comboIdentity.add_SelectionChangeCommitted($comboIdentity_SelectionChangeCommitted)
 
 $form1.Controls.Add($comboIdentity)
 
 $labelIdentity.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 66
 $labelIdentity.Location = $System_Drawing_Point
 $labelIdentity.Name = "labelIdentity"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 50
 $labelIdentity.Size = $System_Drawing_Size
 $labelIdentity.TabIndex = 2
 $labelIdentity.Text = "Identity:"
 $labelIdentity.TextAlign = 16
 
 $form1.Controls.Add($labelIdentity)
 
 $labelTitle.DataBindings.DefaultDataSourceUpdateMode = 0
 $labelTitle.Font = New-Object System.Drawing.Font("Trebuchet MS",9.75,1,3,1)
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 52
 $System_Drawing_Point.Y = 13
 $labelTitle.Location = $System_Drawing_Point
 $labelTitle.Name = "labelTitle"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 31
 $System_Drawing_Size.Width = 469
 $labelTitle.Size = $System_Drawing_Size
 $labelTitle.TabIndex = 1
 $labelTitle.Text = "Exchange Automatic Replies Administrator"
 $labelTitle.TextAlign = 16
 
 $form1.Controls.Add($labelTitle)
 
 
 $pbIcon.BackgroundImageLayout = 3
 $pbIcon.DataBindings.DefaultDataSourceUpdateMode = 0
 
 
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 13
 $pbIcon.Location = $System_Drawing_Point
 $pbIcon.Name = "pbIcon"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 32
 $System_Drawing_Size.Width = 32
 $pbIcon.Size = $System_Drawing_Size
 $pbIcon.TabIndex = 0
 $pbIcon.TabStop = $False
 $pbIcon.Image=$logobmp
 
 $form1.Controls.Add($pbIcon)
 
 $InitialFormWindowState = $form1.WindowState
 $form1.add_Load($OnLoadForm_StateCorrection)
 $form1.ShowDialog()| Out-Null
 
 
 
 function GetAutoReplyStatus ($identity) {
 
     try {
         $status=Get-MailboxAutoReplyConfiguration $identity
 
             switch ($status.autoreplystate) {
 
                 enabled { Status-EnabledAutoReply }
                 disabled { Status-DisabledAutoReply }
                 scheduled { Status-ScheduledAutoReply }
 
             }
 
             $dateStartDate.Value=$status.StartTime
             $dateStartTime.Value=$status.StartTime
             $dateEndDate.Value=$status.EndTime
             $dateEndTime.Value=$status.EndTime
             $textInside.Text=$status.InternalMessage
             $textOutside.Text=$status.ExternalMessage
 
 
             switch ($status.ExternalAudience) {
 
                 All { 
                         $checkReplyOutside.Checked = $true
                         $radioAll.Checked = $true
                         $tabPage2.Text = "Outside My Organization (On)"
                     }
                 None {
                         $checkReplyOutside.Checked = $false
                         $radioAll.Checked = $true
                         $radioAll.enabled=$false
                         $radioKnown.enabled=$false
                         $textOutside.Enabled=$false
                         $tabPage2.Text = "Outside My Organization (Off)"
 
                      }
 
                 Known {
                         $checkReplyOutside.Checked = $true
                         $radioKnown.Checked = $true
                         $tabPage2.Text = "Outside My Organization (On)"
                       }
             }
                         
             $buttonOK.Enabled=$true
             $form1.Update()
     }
     catch { ShowPrompt -Message $_ -Buttons OK -Icon Error;return }
 
 }
 
 function Status-DisabledAutoReply {
 
     $radioDoNotSend.enabled=$true
     $radioDoNotSend.checked = $true
     $radioSend.enabled=$true
     $radioSend.Checked=$false
     $checkOnlySend.checked=$false
     $checkOnlySend.enabled=$false
     $labelStartTime.enabled=$false
     $labelEndTime.enabled=$false
     $dateStartDate.enabled=$false
     $dateStartTime.enabled=$false
     $dateEndDate.enabled=$false
     $dateEndTime.enabled=$false
     $labelAutoReply.enabled=$false
     $textInside.enabled=$false
     $textOutside.enabled=$false
     $checkReplyOutside.enabled=$false
     $radioKnown.enabled=$false
     $radioAll.enabled=$false
     $buttonOK.Enabled=$true
     $form1.Update()
  
 }
  
 function Status-EnabledAutoReply {
 
     $radioDoNotSend.enabled=$true
     $radioDoNotSend.checked = $false
     $radioSend.enabled=$true
     $radioSend.Checked=$true
     $checkOnlySend.checked=$false
     $checkOnlySend.enabled=$true
     $labelStartTime.enabled=$false
     $labelEndTime.enabled=$false
     $dateStartDate.enabled=$false
     $dateStartTime.enabled=$false
     $dateEndDate.enabled=$false
     $dateEndTime.enabled=$false
     $labelAutoReply.enabled=$true
     $textInside.enabled=$true
     $textOutside.enabled=$true
     $checkReplyOutside.enabled=$true
     $radioKnown.enabled=$true
     $radioAll.enabled=$true
     $buttonOK.Enabled=$true
     $form1.Update()
     
 }   
 
 function Status-ScheduledAutoReply {
 
     $radioDoNotSend.enabled=$true
     $radioDoNotSend.checked = $false
     $radioSend.enabled=$true
     $radioSend.Checked=$true
     $checkOnlySend.checked=$true
     $checkOnlySend.enabled=$true
     $labelStartTime.enabled=$true
     $labelEndTime.enabled=$true
     $dateStartDate.enabled=$true
     $dateStartTime.enabled=$true
     $dateEndDate.enabled=$true
     $dateEndTime.enabled=$true
     $labelAutoReply.enabled=$true
     $textInside.enabled=$true
     $textOutside.enabled=$true
     $checkReplyOutside.enabled=$true
     $radioKnown.enabled=$true
     $radioAll.enabled=$true
     $buttonOK.Enabled=$true
     $form1.Update()
     
 }   
 
 Function ShowPrompt {
     param ($Message,$Title=$Script:ScriptTitle,$Buttons,$Icon,$DefaultButton="button1")
     return [System.Windows.Forms.MessageBox]::Show($Message,$Title,$Buttons,$Icon,$DefaultButton)
 }
 
 Function Hide-PowerShell {
         $null = $showWindowAsync::ShowWindowAsync((Get-Process -PID $pid).MainWindowHandle, 0)
 }
 
 $script:showWindowAsync = Add-Type -MemberDefinition @"
 [DllImport("user32.dll")]
 public static extern bool ShowWindowAsync(IntPtr hWnd, int nCmdShow);
 "@ -name Win32ShowWindowAsync -namespace Win32Functions -PassThru
 
 
 [string]$logob64=@"
 iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAABmJLR0QA/wD/AP+gvaeTAAAACXBI
 WXMAAAsTAAALEwEAmpwYAAAAB3RJTUUH3AsVFgcLeQdwxQAAAB1pVFh0Q29tbWVudAAAAAAAQ3Jl
 YXRlZCB3aXRoIEdJTVBkLmUHAAABsklEQVRYw+2XS0sCURiG31M2OqZol8EobNEiKGjhONEvCNq2
 62K1iv5Bi4KsQfoH0Toic9GinS1dBBGl7cKuUF5IJHIodTSH0yKUBMVFHRXy25zNMO/De16+7ztE
 0zQ0strQ4NLV+iB1sUV/K2Idd5OmdaAF0AJoAbQAmr8VF4sfnILeNoFc4hzZ55P6O8D1OsrOugPk
 EmdlZ92vQI0FoMYC/zCElWb5X+wINQFu4wrd9IaQyRUAAF0mPeQ5EXbBRJg7EI6mqOy7Kon3mA2Q
 50UMdPPIPvkpU4Bq4v3WDnzceVFQHthloJL49qIEoVMj6Zs9WkjH2YUwHE3RzcMQ1Pz3mi5YDPC4
 JNisPAEA0+gyYbG86gDgOvJGZd9VSRwAkoqKlZ1TAKj54+P1SfKrPrDrD5eJ130ark6PwWLkGteI
 7IKJyPMi3TgIQcnkAQA2Kw+PSwoKFoPEEoD8fBs+vrxTtzeI9+xnKYjbC+PBHj4vpe+PqJaOsH0Z
 DfWZiXtWhFGvKwVxbf/C+ZrlLs0jS4QTnGwdqNaGi06wuI6K03C430I2ZhwwcO1lTiQV9fKvAb4A
 EHe5VK0uOSAAAAAASUVORK5CYII=
 "@
 
 $logostream=[System.IO.MemoryStream][System.Convert]::FromBase64String($logob64)
 $logobmp=[System.Drawing.Bitmap][System.Drawing.Image]::FromStream($logostream)
 $iconhandle=$logobmp.GetHicon()
 $icon=[System.Drawing.Icon]::FromHandle($iconhandle)
 
 
 [string]$Insideb64=@"
 iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAABmJLR0QA/wD/AP+gvaeTAAAACXBI
 WXMAAAsTAAALEwEAmpwYAAAAB3RJTUUH3AsXFRw6KaDNtwAAAB1pVFh0Q29tbWVudAAAAAAAQ3Jl
 YXRlZCB3aXRoIEdJTVBkLmUHAAAEnElEQVRIx5WVTWyUVRSGn/v9TKedb9optR0V0mIrFAIoqKUx
 NDGoCYSwcisGE4kLF8YdOzfqUk2Ixg3RpDEkaqILEyVqJKjxh5rW/oCVAi1lBjo/TOl8M/PN93eP
 Cwi60Lac5F3c5Jz73vec9+QqEWE9EccxtVqNIAiI4xjP8+jt7cU0zVXrrPVe3mw2mZiYoFQqied5
 zM3Ncfz4ceU4zqq1aj0KLly4wNTUlNRqNcrlMtVqFRFh3759HD58WK2boFqtUqlUmJ2dlUKhgOu6
 XLlyhcHBQQzDwHVdHMehv7+fwcFBzpw5QxAEHDlyRCWTybVbtLCwwMTEhARBQKFQoFwuk8/nGRgY
 oKuri/379+M4abTWaB2TTqc5f/48Wuv1zWBsbEyCIGB+fh7Xdenv7+fQoUNMT0/jeR62baN1fBux
 JpPpZK0W3yVwXZfZ2VmGhoY4cOAAbW1txLFGdIzv+0xOThJFEUoptNY0m03m56/gOA5KqbUJisUi
 juMwMjKCYRhoHSOi0VrT3t6OiNx5uUmsNYVCgXqtzs4dO7FMa22CTCZDd3c3vu/T0tJCHOs7vdbk
 cjm01kxPz6DjmFq9zrVr13DSDolkAtQ6XKS1ZnR0VCzLYvv2bcSxJggCrl5dZO7qHOneDKZWbH1w
 KyNP7mOhkuPnq1Pcihc5OPCM2r3psdUVGIaBiDA+Pg7A8vIyruvSaDSwMzZRZ8il5UVe3PsC6TaH
 pfkG0tjD0I4NnDjzjnx45OP/1GH8+1AJNpPKjOAH0NHRw9Gjxzh4+Hl+HO/l9Eed2EYHv1ye5Jvp
 Kb48a1OsNNEitGXTnPjs/f+0k4qimIYf8v6nY7KyfB8RKao3Znjt2C5SKYf3Rs+RLyuwLPp7IdHZ
 Sq7Ywd7dXdwoFXhooITvR5heguEte5VpGLQkDEyleKDLxmr4IRcXy9wsCspOIVqw2zfx3fcXiUjy
 59wKvVt66cu2kExajF8Xhh/tJGEatBppxs7Z3FwOUYbN2fG8pJI2G+9P4SQMXjq4QVk3Cit8MPqD
 ZHuG8OPbKhMtDj9NLlBcytG9+QH67k/itFl8/cctRob7uJyv07exnZ6sQ082hYhCAUqBUoqmL+SX
 QkQEC9PCSG+gEYJhgGihGcbQ2o1pVdmzvYvFksdcIeSp4c1cX/LRAhcvVf/Xmv0b0+RLdWItGE4y
 wRMPdas4DBEtBFGEH4Q0Qw+tltmYbeX32QqPP7KJxZyPVxP8+uoo3orx/QgEjJ4NbTx3YCtLN6e4
 ls9xq9bAbXi49Tq1aoWZiyVGdnXxyRe/89d8CWUYayKMhIYXIMg/ixaEEef+WOD7ny5JsVCnsuJh
 ErLl4SxDu/vUtoEeXj85I+Wok0xHJw2vSrNRI2lBFEMcCVoEUQYPZlPklqp8/uawUuv9Mm9vuzDy
 8reyeWCQulflt1+nMEwDyzSxEyamYWBaJq0tNgnb5vS7Tw3cEwHA2fEbvHFyUpKZLGF1iVNvPa24
 46A7q4VSPKtQ37U7FojIPUFrTRDGvH1qRg69+pWslW9wj6GUwrYMXnluUK0r/15bdHceIqxUAzo7
 WlbN+xtBXrZJwlIuCAAAAABJRU5ErkJggg==
 "@
 
 $Insidestream=[System.IO.MemoryStream][System.Convert]::FromBase64String($Insideb64)
 $Insidebmp=[System.Drawing.Bitmap][System.Drawing.Image]::FromStream($Insidestream)
 $Insideiconhandle=$Insidebmp.GetHicon()
 $Insideicon=[System.Drawing.Icon]::FromHandle($Insideiconhandle)
 
 [string]$Outsideb64=@"
 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
 AABJRU5ErkJggg==
 "@
 
 $Outsidestream=[System.IO.MemoryStream][System.Convert]::FromBase64String($Outsideb64)
 $Outsidebmp=[System.Drawing.Bitmap][System.Drawing.Image]::FromStream($Outsidestream)
 $Outsideiconhandle=$Outsidebmp.GetHicon()
 $Outsideicon=[System.Drawing.Icon]::FromHandle($Outsideiconhandle)
 
 if (!(Test-Path $env:ExchangeInstallPath\bin\RemoteExchange.ps1)) { ShowPrompt -Message "This application requires the Exchange Management Shell to be installed on this computer. Please install the Exchange Management Tools from the Microsoft Exchange Server installation media." -icon Error -buttons OK;break }
 try {
     .$env:ExchangeInstallPath\bin\RemoteExchange.ps1 | Out-Null
     Connect-ExchangeServer -auto | Out-Null
 }
 catch { ShowPrompt -Message $_ -Buttons OK -Icon Error; break }
 
 GenerateForm
 Get-PSSession | Remove-PSSession -Confirm:$false
 Exit-PSSession
`

