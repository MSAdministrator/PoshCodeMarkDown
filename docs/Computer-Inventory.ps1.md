---
Author: assaf miron
Publisher: 
Copyright: 
Email: 
Version: 15.75
Encoding: ascii
License: cc0
PoshCode ID: 596
Published Date: 2009-09-20t23
Archived Date: 2016-06-18t09
---

# computer inventory - 

## Description

script creates form and retrieves remote (and local) computer inventory.

## Comments



## Usage



## TODO



## script

`get-reg`

## Code

`#
 #
 #
 #
 
 [void][System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms")
 [void][System.Reflection.Assembly]::LoadWithPartialName("System.Drawing")
 
 $LogFile = "C:\Monitoring\Test-Monitor.csv"
 If((Test-Path ($LogFile.Substring(0,$logFile.LastIndexof("\")))) -eq $False)
 { 
 	New-Item ($LogFile.Substring(0,$logFile.LastIndexof("\"))) -Type Directory
 }
 
 
 #~~< Form1 >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $Form1 = New-Object System.Windows.Forms.Form
 $Form1.AutoSize = $TRUE
 $Form1.ClientSize = New-Object System.Drawing.Size(522, 404)
 $Form1.StartPosition = [System.Windows.Forms.FormStartPosition]::CenterScreen
 $Form1.Text = "Computer Inventory"
 #~~< Panel1 >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $Panel1 = new-object System.Windows.Forms.Panel
 $Panel1.Dock = [System.Windows.Forms.DockStyle]::Fill
 $Panel1.Location = new-object System.Drawing.Point(0, 24)
 $Panel1.Size = new-object System.Drawing.Size(522, 380)
 $Panel1.TabIndex = 20
 $Panel1.Text = ""
 #~~< btnRun >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $btnRun = New-Object System.Windows.Forms.Button
 $btnRun.Enabled = $FALSE
 $btnRun.Location = New-Object System.Drawing.Point(431, 30)
 $btnRun.Size = New-Object System.Drawing.Size(75, 23)
 $btnRun.TabIndex = 2
 $btnRun.Text = "Run"
 $btnRun.UseVisualStyleBackColor = $TRUE
 $btnRun.add_Click({ RunScript($btnRun) })
 #~~< Label1 >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $Label1 = New-Object System.Windows.Forms.Label
 $Label1.AutoSize = $False#$TRUE
 $Label1.Location = New-Object System.Drawing.Point(12, 31)
 $Label1.Size = New-Object System.Drawing.Size(163, 13)
 $Label1.TabIndex = 15
 $Label1.Text = "File containing Computer Names"
 #~~< TextBox1 >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $TextBox1 = New-Object System.Windows.Forms.TextBox
 $TextBox1.Location = New-Object System.Drawing.Point(177, 30)
 $TextBox1.Size = New-Object System.Drawing.Size(161, 20)
 $TextBox1.TabIndex = 0
 $TextBox1.Text = ""
 #~~< btnBrowse >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $btnBrowse = New-Object System.Windows.Forms.Button
 $btnBrowse.Location = New-Object System.Drawing.Point(347, 30)
 $btnBrowse.Size = New-Object System.Drawing.Size(75, 23)
 $btnBrowse.TabIndex = 1
 $btnBrowse.Text = "Browse"
 $btnBrowse.UseVisualStyleBackColor = $TRUE
 $btnBrowse.add_Click({ BrowseFile($btnBrowse) })
 #~~< DataGridView1 >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $DataGridView1 = new-object System.Windows.Forms.DataGridView
 $DataGridView1.Anchor = ([System.Windows.Forms.AnchorStyles]([System.Windows.Forms.AnchorStyles]::Top -bor [System.Windows.Forms.AnchorStyles]::Bottom -bor [System.Windows.Forms.AnchorStyles]::Left -bor [System.Windows.Forms.AnchorStyles]::Right))
 $DataGridView1.ColumnHeadersHeightSizeMode = [System.Windows.Forms.DataGridViewColumnHeadersHeightSizeMode]::AutoSize
 $DataGridView1.Location = New-Object System.Drawing.Point(12, 59)
 $DataGridView1.Size = New-Object System.Drawing.Size(497, 280)
 $DataGridView1.TabIndex = 4
 $DataGridView1.ClipboardCopyMode = [System.Windows.Forms.DataGridViewClipboardCopyMode]::Disable
 $DataGridView1.Text = ""
 #~~< ProgressBar1 >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $ProgressBar1 = New-Object System.Windows.Forms.ProgressBar
 $ProgressBar1.Anchor = ([System.Windows.Forms.AnchorStyles] ([System.Windows.Forms.AnchorStyles]::Bottom -bor [System.Windows.Forms.AnchorStyles]::Left -bor [System.Windows.Forms.AnchorStyles]::Right ))
 $ProgressBar1.Location = New-Object System.Drawing.Point(12, 345)
 $ProgressBar1.Size = New-Object System.Drawing.Size(410, 23)
 $ProgressBar1.TabIndex = 5
 $ProgressBar1.Text = ""
 #~~< btnExit >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $btnExit = New-Object System.Windows.Forms.Button
 $btnExit.Anchor = ([System.Windows.Forms.AnchorStyles]([System.Windows.Forms.AnchorStyles]::Bottom -bor [System.Windows.Forms.AnchorStyles]::Right))
 $btnExit.DialogResult = [System.Windows.Forms.DialogResult]::Cancel
 $btnExit.Location = New-Object System.Drawing.Point(431, 345)
 $btnExit.Size = New-Object System.Drawing.Size(78, 23)
 $btnExit.TabIndex = 3
 $btnExit.Text = "Exit"
 $btnExit.UseVisualStyleBackColor = $TRUE
 $btnExit.add_Click({ CloseForm($btnExit) })
 #~~< MenuStrip1 >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $MenuStrip1 = new-object System.Windows.Forms.MenuStrip
 $MenuStrip1.Location = new-object System.Drawing.Point(0, 0)
 $MenuStrip1.Size = new-object System.Drawing.Size(292, 24)
 $MenuStrip1.TabIndex = 6
 $MenuStrip1.Text = "MenuStrip1"
 #~~< FileToolStripMenuItem >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $FileToolStripMenuItem = new-object System.Windows.Forms.ToolStripMenuItem
 $FileToolStripMenuItem.Size = new-object System.Drawing.Size(35, 20)
 $FileToolStripMenuItem.Text = "File"
 #~~< OpenLogFileToolStripMenuItem >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $OpenLogFileToolStripMenuItem = new-object System.Windows.Forms.ToolStripMenuItem
 $OpenLogFileToolStripMenuItem.Size = new-object System.Drawing.Size(152, 22)
 $OpenLogFileToolStripMenuItem.Text = "Open Log File"
 $OpenLogFileToolStripMenuItem.add_Click({Open-file($OpenLogFileToolStripMenuItem)})
 #~~< ExitToolStripMenuItem >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $ExitToolStripMenuItem = new-object System.Windows.Forms.ToolStripMenuItem
 $ExitToolStripMenuItem.Size = new-object System.Drawing.Size(152, 22)
 $ExitToolStripMenuItem.Text = "Exit"
 $ExitToolStripMenuItem.add_Click({CloseForm($ExitToolStripMenuItem)})
 $FileToolStripMenuItem.DropDownItems.AddRange([System.Windows.Forms.ToolStripItem[]](@($OpenLogFileToolStripMenuItem, $ExitToolStripMenuItem)))
 #~~< HelpToolStripMenuItem >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $HelpToolStripMenuItem = new-object System.Windows.Forms.ToolStripMenuItem
 $HelpToolStripMenuItem.Size = new-object System.Drawing.Size(40, 20)
 $HelpToolStripMenuItem.Text = "Help"
 #~~< AboutToolStripMenuItem >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $AboutToolStripMenuItem = new-object System.Windows.Forms.ToolStripMenuItem
 $AboutToolStripMenuItem.Size = new-object System.Drawing.Size(152, 22)
 $AboutToolStripMenuItem.Text = "About"
 $AboutToolStripMenuItem.add_Click({Show-About($AboutToolStripMenuItem)})
 #~~< HowToToolStripMenuItem >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $HowToToolStripMenuItem = new-object System.Windows.Forms.ToolStripMenuItem
 $HowToToolStripMenuItem.Size = new-object System.Drawing.Size(152, 22)
 $HowToToolStripMenuItem.Text = "How To?"
 $HowToToolStripMenuItem.add_Click({Show-HowTo($HowToToolStripMenuItem)})
 $HelpToolStripMenuItem.DropDownItems.AddRange([System.Windows.Forms.ToolStripItem[]](@($AboutToolStripMenuItem, $HowToToolStripMenuItem)))
 $MenuStrip1.Items.AddRange([System.Windows.Forms.ToolStripItem[]](@($FileToolStripMenuItem, $HelpToolStripMenuItem)))
 $Panel1.Controls.Add($MenuStrip1)
 $Panel1.Controls.Add($btnRun)
 $Panel1.Controls.Add($Label1)
 $Panel1.Controls.Add($TextBox1)
 $Panel1.Controls.Add($btnBrowse)
 $Panel1.Controls.Add($ProgressBar1)
 $Panel1.Controls.Add($btnExit)
 $Panel1.Controls.Add($DataGridView1)
 $Panel1.Controls.Add($Menu)
 $Form1.Controls.Add($Panel1)
 #~~< Ping1 >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $Ping1 = New-Object System.Net.NetworkInformation.Ping
 #~~< OpenFileDialog1 >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $OpenFileDialog1 = New-Object System.Windows.Forms.OpenFileDialog
 $OpenFileDialog1.Filter = "Text Files|*.txt|CSV Files|*.csv|All Files|*.*"
 $OpenFileDialog1.InitialDirectory = "C:"
 $OpenFileDialog1.Title = "Open Computers File"
 #~~< objNotifyIcon >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 $objNotifyIcon = New-Object System.Windows.Forms.NotifyIcon 
 $objNotifyIcon.Icon = "D:\Assaf\Scripts\Icons\XP\people15.ico"
 $objNotifyIcon.BalloonTipIcon = "Info" 
 
 
 
 function out-DataTable
 {
 	$dt = New-Object Data.datatable
 	$First = $TRUE	
 
 	foreach ($item in $Input)
 	{
 		$DR = $DT.NewRow()
 		$Item.PsObject.get_properties() | foreach {
 			if ($first)
 			{
 				$Col = New-Object Data.DataColumn
 				$Col.ColumnName = $_.Name.ToString()
 			$DT.Columns.Add($Col) }
 			if ($_.value -eq $null)
 			{
 				$DR.Item($_.Name) = "[empty]"
 			}
 			elseif ($_.IsArray) {
 				$DR.Item($_.Name) = [string]::Join($_.value, ";")
 			}
 			else
 			{
 				$DR.Item($_.Name) = $_.value
 			}
 		}
 		$DT.Rows.Add($DR)
 		$First = $FALSE
 	}
 		
 	return @(, ( $dt ))
 		
 }
 
 function Join-Data
 {
 	foreach ($item in $Input)
 	{
 		$Item.PsObject.get_properties() | foreach{
 			if ($_.value -eq $null)
 			{
 				$DataObject | Add-Member noteproperty $_.Name "[empty]"
 			}
 			elseif ($_.IsArray) {
 				$DataObject | Add-Member noteproperty $_.Name [string]::Join($_.value, ";")
 			}
 			elseif ($objName -ne "") {
 				$DataObject | Add-Member noteproperty $objName $Item
 			}
 			else
 			{
 				$DataObject | Add-Member noteproperty $_.Name $_.value -Force
 			}
 		}
 	}
 	
 	return @(,$DataObject)
 }
 
 function Get-Reg {
 	param(
 		$Hive,
 		$Key,
 		$Value,
 	)
 	$reg = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey($Hive, $RemoteComputer)
 	$regKey= $reg.OpenSubKey($Key)
 }
 
 function Get-WMIItem {
 	param(
 		$Class,
 		$Item,
 	)
 	{
 		gwmi -Class $Class -ComputerName $RemoteComputer -Filter $Filter -Property $Item | Select $Item
 	}
 	{gwmi -ComputerName $RemoteComputer -Query $Query | select $Item}
 }
 
 function Show-NotifyIcon {
 	param(
 		$Title,
 		$Text
 	)
 		$objNotifyIcon.BalloonTipTitle = $Title
 		$objNotifyIcon.BalloonTipText = $Text
 		$objNotifyIcon.Visible = $TRUE 
 		$objNotifyIcon.ShowBalloonTip(10000)
 }
 
 
 
 function Main
 {
 	[System.Windows.Forms.Application]::EnableVisualStyles()
 	[System.Windows.Forms.Application]::Run($Form1)
 }
 
 
 
 
 function BrowseFile($object)
 {
 	$OpenFileDialog1.showdialog()
 	$TextBox1.Text = $OpenFileDialog1.FileName
 	$btnRun.Enabled = $TRUE
 }
 
 function Open-File( $object ){
 	if(Test-Path $LogFile){
 		Invoke-Item $LogFile
 	}
 }
 
 function Show-PopUp
 {
 	param(
 		$PopupTitle,
 		$PopupText
 		)
 	#~~< PopupForm >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 	$PopupForm = New-Object System.Windows.Forms.Form
 	$PopupForm.ClientSize = New-Object System.Drawing.Size(381, 356)
 	$PopupForm.ControlBox = $false
 	$PopupForm.ShowInTaskbar = $false
 	$PopupForm.StartPosition = [System.Windows.Forms.FormStartPosition]::CenterParent
 	$PopupForm.Text = $PopupTitle
 	#~~< PopupColse >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 	$PopupColse = New-Object System.Windows.Forms.Button
 	$PopupColse.Anchor = ([System.Windows.Forms.AnchorStyles]([System.Windows.Forms.AnchorStyles]::Bottom -bor [System.Windows.Forms.AnchorStyles]::Left -bor [System.Windows.Forms.AnchorStyles]::Right))
 	$PopupColse.DialogResult = [System.Windows.Forms.DialogResult]::Cancel
 	$PopupColse.Location = New-Object System.Drawing.Point(137, 321)
 	$PopupColse.Size = New-Object System.Drawing.Size(104, 23)
 	$PopupColse.TabIndex = 0
 	$PopupColse.Text = "Close"
 	$PopupColse.UseVisualStyleBackColor = $true
 	$PopupForm.AcceptButton = $PopupColse
 	$PopupForm.CancelButton = $PopupColse
 	#~~< PopupHeader >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 	$PopupHeader = New-Object System.Windows.Forms.Label
 	$PopupHeader.Font = New-Object System.Drawing.Font("Calibri", 15.75, ([System.Drawing.FontStyle]([System.Drawing.FontStyle]::Bold -bor [System.Drawing.FontStyle]::Italic)), [System.Drawing.GraphicsUnit]::Point, ([System.Byte](0)))
 	$PopupHeader.Location = New-Object System.Drawing.Point(137, 9)
 	$PopupHeader.Size = New-Object System.Drawing.Size(104, 23)
 	$PopupHeader.TabIndex = 2
 	$PopupHeader.Text = $PopupTitle
 	$PopupHeader.TextAlign = [System.Drawing.ContentAlignment]::MiddleCenter
 	#~~< PopUpTextArea >~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 	$PopUpTextArea = New-Object System.Windows.Forms.Label
 	$PopUpTextArea.Anchor = ([System.Windows.Forms.AnchorStyles]([System.Windows.Forms.AnchorStyles]::Top -bor [System.Windows.Forms.AnchorStyles]::Bottom -bor [System.Windows.Forms.AnchorStyles]::Left -bor [System.Windows.Forms.AnchorStyles]::Right))
 	$PopUpTextArea.Location = New-Object System.Drawing.Point(12, 15)
 	$PopUpTextArea.Size = New-Object System.Drawing.Size(357, 265)
 	$PopUpTextArea.TabIndex = 1
 	$PopUpTextArea.Text = $PopupText
 	$PopUpTextArea.TextAlign = [System.Drawing.ContentAlignment]::MiddleLeft
 	$PopupForm.Controls.Add($PopupHeader)
 	$PopupForm.Controls.Add($PopUpTextArea)
 	$PopupForm.Controls.Add($PopupColse)
 	
 	$PopupForm.Add_Shown({$PopupForm.Activate()})  
 	[void]$PopupForm.showdialog() 
 
 }
 
 function Show-About( $object ){
 	$AboutText = @("
 	Script Title:	            Remote Computer Inventory`n
 	Script Author:              Assaf Miron`n
 	Script Description:         Collects Remote Computer Data Using WMI and Registry Access Outputs all information to a Data Grid Form and to a CSV Log File.`n
 	
 	Log File Name:	$LogFile")
 	
 	Show-Popup -PopupTitle "About" -PopupText $AboutText
 }
 
 function Show-HowTo( $object ){
 	$HowToText = @("
 	1. Click on the Browse Button and select a TXT or a CSV File Containing Computer Names`n
 	2. After File is Selected click on the Run Button.`n
 	3. You will see a Notify Icon with the Coresponding Text.`n
 	4. The Script has begon collecting Remote Computer Inventory!`n
 	`nWhen The script is Done you will see a Popup Message and all data will be presented in the DataGrid.`n
 	** Because Poweshell works only in MTA mode there is no Option Copying the Data off the DataGrid...`n
 	5. All Data will be Exported to a Log File Located Here: $LogFile")
 	
 	Show-Popup -PopupTitle "How To?" -PopupText $HowToText
 }
 
 function CloseForm($object)
 {
 	$Form1.Close()
 }
 
 function RunScript($object)
 {
 $arrComputers = Get-Content -path $textBox1.Text -encoding UTF8
 
 $AllComputers = @()
 
 if(($arrComputers -is [array]) -eq $FALSE) { $ProgressBar1.Maximum = 1 }
 else { $ProgressBar1.Maximum = $arrComputers.Count }
 $ProgressBar1.Minimum = 0
 $ProgressBar1.Value = 0
 
 foreach ($strComputer in $arrComputers)
 	{ 
 		if($Ping1.Send($strComputer).Status -eq "Success"){
 		Show-NotifyIcon -Title "Retriving Computer Information" -Text "Scanning $strComputer For Hardware Data" 
 
 		$ComputerDet = Get-WMIItem -Class "Win32_computersystem" -RemoteComputer $strComputer -Item Caption,Domain,SystemType,Manufacturer,Model,NumberOfProcessors,TotalPhysicalMemory,UserName
 		
 		{	
 
 			if($ComputerDet.TotalPhysicalMemory -ge 1GB){
 
 			$CPUName = Get-WMIItem -Class "Win32_Processor" -RemoteComputer $strComputer -Item Name
 			$arrCPUNames = @() 
 			foreach($CPU in $CPUName){
 				}
 
 			$OS = Get-WMIItem -Class "win32_operatingsystem" -RemoteComputer $strComputer -Item Caption,csdversion
 
 			$ChassisType = Get-WMIItem -Class Win32_SystemEnclosure -RemoteComputer $strComputer -Item ChassisTypes 
 			switch ($ChassisType.ChassisTypes) {
 					1 {$ChassisType = "Other"}
 					2 {$ChassisType = "Unknown"}
 					3 {$ChassisType = "Desktop"}
 					4 {$ChassisType = "Low Profile Desktop"}
 					5 {$ChassisType = "Pizza Box"}
 					6 {$ChassisType = "Mini Tower"}
 					7 {$ChassisType = "Tower"}
 					8 {$ChassisType = "Portable"}
 					9 {$ChassisType = "Laptop"}
 					10 {$ChassisType = "Notebook"}
 					11 {$ChassisType = "Handheld"}
 					12 {$ChassisType = "Docking Station"}
 					13 {$ChassisType = "All-in-One"}
 					14 {$ChassisType = "Sub-Notebook"}
 					15 {$ChassisType = "Space Saving"}
 					16 {$ChassisType = "Lunch Box"}
 					17 {$ChassisType = "Main System Chassis"}
 					18 {$ChassisType = "Expansion Chassis"}
 					19 {$ChassisType = "Sub-Chassis"}
 					20 {$ChassisType = "Bus Expansion Chassis"}
 					21 {$ChassisType = "Peripheral Chassis"}
 					22 {$ChassisType = "Storage Chassis"}
 					23 {$ChassisType = "Rack Mount Chassis"}
 					24 {$ChassisType = "Sealed- PC"}
 					default {$ChassisType = "Unknown"}
 				}
 
 			$AUOptions = Get-Reg -Hive LocalMachine -Key "SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update" -Value "AUOptions"
 			
 			$AUDay = Get-Reg -Hive LocalMachine -Key "SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update" -Value "ScheduledInstallDay"
 			
 			$AUTime = Get-Reg -Hive LocalMachine -Key "SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update" -Value "ScheduledInstallTime"
 				$AUOptions = Get-Reg -Hive LocalMachine -Key "SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -Value "AUOptions"
 				$AUDay = Get-Reg -Hive LocalMachine -Key "SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -Value "ScheduledInstallDay"
 				$AUTime = Get-Reg -Hive LocalMachine -Key "SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -Value "ScheduledInstallTime"
 				}
 				1 {$AUClient = "Automatic Updates is Turnd off."}
 				2 {$AUClient = "Notify for download and notify for install "}
 				3 {$AUClient = "Auto download and notify for install "}
 				4 {
 					{
 					0 {$InstDay = "Every Day"}
 					1 {$InstDay = "Sunday"}
 					2 {$InstDay = "Monday"}
 					3 {$InstDay = "Tuesday"}
 					4 {$InstDay = "Wensday"}
 					5 {$InstDay = "Thursday"}
 					6 {$InstDay = "Friday"}
 					7 {$InstDay = "Saturday"}
 					}
 					if ($AUTime -le 12) { $AUTime = $AUTime.ToString() + " AM" } else { $AUTime = ($AUTime -12) + " PM" }
 						$AUClient = "Auto download and schedule the install - "+$InstDay+" "+$AUTime}
 			}
 
 			$AvialableMem = Get-WMIItem -Class "Win32_PerfFormattedData_PerfOS_Memory" -RemoteComputer $strComputer -Item "AvailableMBytes"
 			
 			$DiskQueue = Get-WMIItem -Class "Win32_PerfFormattedData_PerfDisk_LogicalDisk" -RemoteComputer $strComputer -Item CurrentDiskQueueLength
 			$QueueLength = Get-WMIItem -Class "Win32_PerfFormattedData_PerfNet_ServerWorkQueues" -RemoteComputer $strComputer -Item QueueLength
 			$Processor = Get-WMIItem -Class "Win32_PerfFormattedData_PerfOS_Processor" -RemoteComputer $strComputer -Item PercentProcessorTime
 			
 			if($AvialableMem.AvailableMBytes -lt 4) { $intHealth += 1; $strHealth += "Low Free Memory;" }
 			if($DiskQueue.CurrentDiskQueueLength -gt 2) { $intHealth += 1; $strHealth += "High Disk Queue;" }
 			if($QueueLength.QueueLength -gt 4) { $intHealth += 1; $strHealth += "Long Disk Queue;" }
 			if($Processor.PercentProcessorTime -gt 90) { $intHealth += 1; $strHealth += "Processor Usage Over 90%;" }
 			if($intHealth -gt 1) { $ComputerTotalHealth = "UnHealthy, " + $strHealth } else { $ComputerTotalHealth = "Healthy" }
 
 
 			$DriveInfo = Get-WMIItem -Class "Win32_LogicalDisk" -RemoteComputer $strComputer -Item Caption,Size,FreeSpace
 			foreach($DRSize in $DriveInfo)
 					if($DRSize.Size -ge 1GB){
 					if($DRSize.FreeSpace -ge 1GB){
 				}
 			$arrDiskDrives = @() 
 			$arrDiskSize = @() 
 			$arrDiskFreeSpace = @() 
 			foreach($Drive in $DriveInfo){
 				$arrDiskDrives = $Drive.Caption+";"+$arrDiskDrives
 				$arrDiskSize = $Drive.Size+";"+$arrDiskSize
 				$arrDiskFreeSpace = $Drive.FreeSpace+";"+$arrDiskFreeSpace
 				}
 
 			$IPAddress = Get-WmiItem -Class "Win32_NetworkAdapterConfiguration" -Filter "IPEnabled = True" -RemoteComputer $strComputer -Item IPAddress
 			$arrIPAddress = @() 
 			foreach($IP in $IPAddress){
 				$arrIPAddress = $IP.IPAddress[0]+";"+$arrIPAddress
 				}
 
 			$TimeZone = Get-WMIItem -Class "Win32_TimeZone" -RemoteComputer $strComputer -Item Bias,StandardName
 			$TimeZone.Bias = $TimeZone.Bias/60
 
 			$SysRestoreStatus = Get-Reg -Hive LocalMachine -RemoteComputer $strComputer -Key "SOFTWARE\Microsoft\Windows NT\CurrentVersion\SystemRestore" -Value "DisableSR"
 			if ($SysRestoreStatus -eq 0) { $SysRestoreStatus = "Enabled" } else { $SysRestoreStatus = "Disabled" }
 
 			$OfflineFolStatus = Get-Reg -Hive LocalMachine -RemoteComputer $strComputer -Key "SOFTWARE\Microsoft\Windows\CurrentVersion\NetCache" -Value "Enabled"
 			if ($OfflineFolStatus -eq 1) { $OfflineFolStatus = "Enabled" } else { $OfflineFolStatus = "Disabled" }
 
 			$Printers = Get-WMIItem -Class "Win32_Printer" -RemoteComputer $strComputer -Item Name,PortName,Caption
 			$arrPrinters = @() 
 			foreach($Printer in $Printers){
 				$arrPrinters = $Printer.Name+"("+$Printer.PortName+");"+$arrPrinters
 				}
 
 			$BIOSSN = Get-WMIItem -Class "Win32_Bios" -RemoteComputer $strComputer -Item SerialNumber
 
 			$NetDrives = Get-WMIItem -Query "Select * From Win32_LogicalDisk Where DriveType = 4" -RemoteComputer $strComputer -Item DeviceID,ProviderName
 			$arrNetDrives = @() 
 			foreach($NetDrive in $NetDrives){
 				$arrNetDrives = $NetDrive.DeviceID+"("+$NetDrive.ProviderName+");"+$arrNetDrives
 				}
 
 			$AVParentServer = Get-Reg -Hive LocalMachine -RemoteComputer $strComputer -Key "SOFTWARE\Intel\LANDesk\VirusProtect6\CurrentVersion" -Value "Parent"
 			$VirusDefFile = "C:\Program Files\Common Files\Symantec Shared\VirusDefs\definfo.dat"
 			If(Test-Path $VirusDefFile){
 				$AVDefs = Get-Content $VirusDefFile | where { $_ -match "CurDefs" }
 				$AVDefs = $AVDefs.Substring(8)
 				$AVDefs = [datetime]($AVDefs.Substring(5,1)  + "/" + $AVDefs.Substring(6,2) + "/" + $AVDefs.substring(0,4))
 			}
 			Else { $AVDefs = "" }
 
 			$HotFixes = Get-WMIItem -Class "Win32_QuickFixEngineering" -RemoteComputer $strComputer -Item Description,HotFixID,ServicePackInEffect
 			$arrHotFixes = @()
 			foreach($Fix in $HotFixes){
 				if($Fix.Description -eq ""){
 					if($Fix.HotFixID -eq "File 1"){	$arrHotFixes = $Fix.ServicePackInEffect+";"+$arrHotFixes }
 					else { $arrHotFixes =$Fix.HotFixID+";"+$arrHotFixes }
 				}
 				else { $arrHotFixes = $Fix.Description+";"+$arrHotFixes }
 			}
 
 			$RDPStatus = Get-Reg -Hive LocalMachine -remoteComputer $strComputer -Key "SYSTEM\CurrentControlSet\Control\Terminal Server" -Value "fAllowToGetHelp"
 			if($RDPStatus -eq 0) {$RDPStatus = "Enabled" } else {$RDPStatus = "Disabled" }
 
 			$RAStatus = Get-Reg -Hive LocalMachine -remoteComputer $strComputer -Key "SYSTEM\CurrentControlSet\Control\Terminal Server" -Value "fDenyTSConnections"
 			if($RAStatus -eq 1) {$RAStatus = "Enabled" } else {$RAStatus = "Disabled" }
 
 			Show-NotifyIcon -Text "Exporting $strComputer Information" -Title "Exporting..."
 			
 			if($ComputerDet -eq $Null){ $ComputerDet = " " }
 			if($ChassisType -eq $Null){ $ChassisType = " " }
 			if($BIOSSN -eq $Null){ $BIOSSN = " " }
 			if($CPUName -eq $Null){ $CPUName = " " }
 			if($AvialableMem -eq $Null){ $AvialableMem = " " }	
 			if($OS -eq $Null){ $OS = " " }
 			if($SP -eq $Null){ $SP = " " }
 			if($IPAddress -eq $Null){ $IPAddress = " " }
 			if($HotFixes -eq $Null){ $HotFixes = " " }
 			if($arrDiskDrives -eq $Null){ $arrDiskDrives=" " }
 			if($arrDiskFreeSpace -eq $Null){ $arrDiskFreeSpace=" " }
 			if($arrDiskSize -eq $Null){ $arrDiskSize=" " }
 			if($RDPStatus -eq $Null){ $RDPStatus = " " }
 			if($RAStatus -eq $Null){ $RAStatus = " " }
 			if($AUClient -eq $Null){ $AUClient = " " }
 			if($AVParentServer -eq $Null){ $AVParentServer = " " }
 			if($AVDefs -eq $Null){ $AVDefs = " " }
 			if($Printers -eq $Null){ $Printers = " " }
 			if($ComputerTotalHealth -eq $Null){ $ComputerTotalHealth = " " }
 
 			$DataObject = New-Object psobject
 			$AllComputers += $DataObject
 			$DataTable = $AllComputers | out-dataTable
 			$DataGridView1.DataSource = $DataTable.psObject.baseobject
 
 			
 			$DataObject | Export-Csv -Encoding OEM -Path $LogFile
 
 		} 
 		}
 			$objNotifyIcon.BalloonTipIcon = "Error" 
 			Show-NotifyIcon -Title "$strComputer is not avialable" -Text "No Ping to $strComputer.`nNo Data was Collected"
 			$objNotifyIcon.BalloonTipIcon = "Info" 
 		  }
 	   
 		$ProgressBar1.PerformStep()
 	  }
 	
 
 
 $objNotifyIcon.Icon = "D:\Assaf\Scripts\Icons\XP\people5.ico"
 $objNotifyIcon.BalloonTipIcon = "Info" 
 
 $MSGObject = new-object -comobject wscript.shell
 $MSGResult = $MSGObject.popup("Script Has Finished Running!",0,"I'm Done",0)
 
 $objNotifyIcon.BalloonTipText = "Done!`nFile Saved in "+$LogFile
 $objNotifyIcon.Visible = $TRUE 
 $objNotifyIcon.ShowBalloonTip(10000)
 
 }
 
 
`

