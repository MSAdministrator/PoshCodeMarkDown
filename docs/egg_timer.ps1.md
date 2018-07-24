---
Author: tojo2000
Publisher: 
Copyright: 
Email: 
Version: 20.25
Encoding: ascii
License: cc0
PoshCode ID: 6311
Published Date: 2016-04-21t19
Archived Date: 2016-10-18t11
---

# egg_timer - 

## Description

a script i submitted for event 10 of the scripting games.  displays a simple windows form that counts down three minutes.  it makes a good example for using windows forms.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function GenerateForm {
   [reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
   [reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 
   $form_main = New-Object System.Windows.Forms.Form
   $reset_button = New-Object System.Windows.Forms.Button
   $label1 = New-Object System.Windows.Forms.Label
   $start_button = New-Object System.Windows.Forms.Button
   $progressBar1 = New-Object System.Windows.Forms.ProgressBar
   $timer1 = New-Object System.Windows.Forms.Timer
   $InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
   $start_button_OnClick = {
     $timer1.Enabled = $true
     $timer1.Start()
     $start_button.Text = 'Countdown Started.'
   }
 
   $reset_button_OnClick = {
     $timer1.Enabled = $false
     $progressBar1.Value = 0
     $start_button.Text = 'Start'
 	$label1.Text = '3:00'
   }
 
   $timer1_OnTick = {
     $progressBar1.PerformStep()
 
     $time = 180 - $progressBar1.Value
     [char[]]$mins = "{0}" -f ($time / 60)
     $secs = "{0:00}" -f ($time % 60)
 
     $label1.Text = "{0}:{1}" -f $mins[0], $secs
   
     if ($progressBar1.Value -eq $progressBar1.Maximum) {
       $timer1.Enabled = $false
       $start_button.Text = 'FINISHED!'
     }
   }
 
   $OnLoadForm_StateCorrection = {
   $form_main.WindowState = $InitialFormWindowState
   }
 
   $System_Drawing_Size = New-Object System.Drawing.Size
   $System_Drawing_Size.Width = 628
   $System_Drawing_Size.Height = 295
   $form_main.MaximumSize = $System_Drawing_Size
 
   $form_main.Text = 'Super Duper Over-engineered Egg Timer'
   $form_main.MaximizeBox = $False
   $form_main.Name = 'form_main'
   $form_main.ShowIcon = $False
   $System_Drawing_Size = New-Object System.Drawing.Size
   $System_Drawing_Size.Width = 628
   $System_Drawing_Size.Height = 295
   $form_main.MinimumSize = $System_Drawing_Size
   $form_main.StartPosition = 1
   $form_main.DataBindings.DefaultDataSourceUpdateMode = 0
   $System_Drawing_Size = New-Object System.Drawing.Size
   $System_Drawing_Size.Width = 612
   $System_Drawing_Size.Height = 259
   $form_main.ClientSize = $System_Drawing_Size
 
   $reset_button.TabIndex = 4
   $reset_button.Name = 'button2'
   $System_Drawing_Size = New-Object System.Drawing.Size
   $System_Drawing_Size.Width = 209
   $System_Drawing_Size.Height = 69
   $reset_button.Size = $System_Drawing_Size
   $reset_button.UseVisualStyleBackColor = $True
 
   $reset_button.Text = 'Reset'
   $reset_button.Font = New-Object System.Drawing.Font("Verdana",12,0,3,0)
 
   $System_Drawing_Point = New-Object System.Drawing.Point
   $System_Drawing_Point.X = 362
   $System_Drawing_Point.Y = 13
   $reset_button.Location = $System_Drawing_Point
   $reset_button.DataBindings.DefaultDataSourceUpdateMode = 0
   $reset_button.add_Click($reset_button_OnClick)
 
   $form_main.Controls.Add($reset_button)
 
   $label1.TabIndex = 3
   $label1.TextAlign = 32
   $System_Drawing_Size = New-Object System.Drawing.Size
   $System_Drawing_Size.Width = 526
   $System_Drawing_Size.Height = 54
   $label1.Size = $System_Drawing_Size
   $label1.Text = '3:00'
   $label1.Font = New-Object System.Drawing.Font("Courier New",20.25,1,3,0)
 
   $System_Drawing_Point = New-Object System.Drawing.Point
   $System_Drawing_Point.X = 45
   $System_Drawing_Point.Y = 89
   $label1.Location = $System_Drawing_Point
   $label1.DataBindings.DefaultDataSourceUpdateMode = 0
   $label1.Name = 'label1'
 
   $form_main.Controls.Add($label1)
 
   $start_button.TabIndex = 2
   $start_button.Name = 'button1'
   $System_Drawing_Size = New-Object System.Drawing.Size
   $System_Drawing_Size.Width = 310
   $System_Drawing_Size.Height = 70
   $start_button.Size = $System_Drawing_Size
   $start_button.UseVisualStyleBackColor = $True
 
   $start_button.Text = 'Start'
   $start_button.Font = New-Object System.Drawing.Font("Verdana",12,0,3,0)
 
   $System_Drawing_Point = New-Object System.Drawing.Point
   $System_Drawing_Point.X = 45
   $System_Drawing_Point.Y = 12
   $start_button.Location = $System_Drawing_Point
   $start_button.DataBindings.DefaultDataSourceUpdateMode = 0
   $start_button.add_Click($start_button_OnClick)
 
   $form_main.Controls.Add($start_button)
 
   $progressBar1.DataBindings.DefaultDataSourceUpdateMode = 0
   $progressBar1.Maximum = 180
   $System_Drawing_Size = New-Object System.Drawing.Size
   $System_Drawing_Size.Width = 526
   $System_Drawing_Size.Height = 87
   $progressBar1.Size = $System_Drawing_Size
   $progressBar1.Step = 1
   $progressBar1.TabIndex = 0
   $System_Drawing_Point = New-Object System.Drawing.Point
   $System_Drawing_Point.X = 45
   $System_Drawing_Point.Y = 146
   $progressBar1.Location = $System_Drawing_Point
   $progressBar1.Style = 1
   $progressBar1.Name = 'progressBar1'
   
   $form_main.Controls.Add($progressBar1)
   
   $timer1.Interval = 1000
   $timer1.add_tick($timer1_OnTick)
 
   $InitialFormWindowState = $form_main.WindowState
   $form_main.add_Load($OnLoadForm_StateCorrection)
   $form_main.ShowDialog()| Out-Null
 
 } 
 
 GenerateForm
`

