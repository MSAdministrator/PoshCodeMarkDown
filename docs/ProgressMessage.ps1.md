---
Author: denis st-pierre
Publisher: 
Copyright: 
Email: 
Version: 1.0.10.0
Encoding: ascii
License: cc0
PoshCode ID: 4020
Published Date: 2013-03-15t17
Archived Date: 2016-04-26t06
---

# progressmessage - 

## Description

creates a persistent popup that stays up while your script does other things.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function ProgressMessage {
 	########################################################################
 	########################################################################
 	
 		[String]$LabelText="Message",
 		[String]$FormTitle="Form Title",
 		[int]$FontSize = 12,
 		[String]$ICOpath=""
 	)
 	
 	If ( Get-Variable -Name ProgMsgForm1 -Scope Global -ErrorAction SilentlyContinue ) {
 	}
 	If ($LabelText -eq ""){
 		Return
 	}
 	
 		
 	[reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
 	[reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 
 	$global:ProgMsgForm1 = New-Object System.Windows.Forms.Form
 	$ProgMsgLabel1 = New-Object System.Windows.Forms.Label
 	$InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
 	$ProgMsgForm1.AutoSize = $True
 	$ProgMsgForm1.AutoSizeMode = 0
 	#$System_Drawing_Size.Height = 113
 	#$System_Drawing_Size.Width = 255
 	$ProgMsgForm1.DataBindings.DefaultDataSourceUpdateMode = 0
 	
 	If ( ($ICOpath -ne "") -and (Test-Path "$ICOpath") ) {
 			$ProgMsgForm1.Icon = [System.Drawing.Icon]::ExtractAssociatedIcon("$ICOpath")
 	}
 	
 	$ProgMsgForm1.Name = "form1"
 	$ProgMsgForm1.Text = "$FormTitle"
 
 	$ProgMsgLabel1.AutoSize = $True
 	$ProgMsgLabel1.DataBindings.DefaultDataSourceUpdateMode = 0
 
 	$System_Drawing_Point = New-Object System.Drawing.Point
 	$ProgMsgLabel1.Location = $System_Drawing_Point
 	$ProgMsgLabel1.Name = "label1"
 	$ProgMsgLabel1.TabIndex = 0
 	$ProgMsgLabel1.Text = "$LabelText"
 	$ProgMsgLabel1.TextAlign = 2
 	$ProgMsgLabel1.Font = New-Object System.Drawing.Font("Microsoft Sans Serif",$FontSize,1,3,1)
 	$ProgMsgForm1.Controls.Add($ProgMsgLabel1)
 
 
 	$InitialFormWindowState = $ProgMsgForm1.WindowState
 	$ProgMsgForm1.add_Load($OnLoadForm_StateCorrection)
 	
  
  
  
 $ProgressObj=ProgressMessage "this is text to display this is text to display this is text to display this is text to display this is text to display this is text to display this is text to display this is text to display this is text to display this is text to display this is text to display this is text to display" "My title"
 Echo "doing other stuff"
 Start-Sleep -Second 2
 Echo "doing other stuff"
 Start-Sleep -Second 2
 
 
 $ProgressObj=ProgressMessage "LEt's see if this works too"
 Echo "doing other stuff"
 Start-Sleep -Second 3
 Echo "doing other stuff"
 Start-Sleep -Second 2
 #$ProgressObj.Close()
 
`

