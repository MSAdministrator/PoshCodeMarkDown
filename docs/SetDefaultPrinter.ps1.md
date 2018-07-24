---
Author: brad blaylock
Publisher: 
Copyright: 
Email: 
Version: 3.2
Encoding: utf-8
License: cc0
PoshCode ID: 2989
Published Date: 2012-10-06t12
Archived Date: 2016-04-27t10
---

# setdefaultprinter.ps1 - 

## Description

set a printer as default  for any user who has ever logged onto any given computer in a domain.

## Comments



## Usage



## TODO



## script

`test-alive`

## Code

`#
 #
 <#
 .SYNOPSIS
     Sets the Default Printer for any user on any machine in active directory.
 .DESCRIPTION
     Search AD for Computername; Select User Account and Printer and make that printer the default
 	printer for that user on that computer.
 .PARAMETER Hostnme
 	This parameter is required and should reflect the computer name (or names) you wish to inventory.
 	This is basically a search criteria because the first thing the script will do is search Active
 	Directory for computer names matching this input.  
 .NOTES
     Author:  	Brad Blaylock
 	Version:	3.2
     Date:    	October 3, 2011
 	Contact:	brad.blaylock@gmail.com
     Details:  	This Script accepts an AD Query (Computername) in the form of an entire computer name
 			or partial computername as in sbcomp* or *2471wk1 and will find any listed computers
 			that match the criteria in AD.  This list will be presented to the user.  The user will 
 			select the computer they are needing, and click [LOAD].  Load will find all user accounts
 			that have ever had a profile on that computer; Load will also list all printers installed
 			on that computer.  The user will then select the user account and printer and click
 			[Set Default].  SetDefault will make the selected printer default for the selected user
 			on the selected computer.  
 				3.0		-If the selected user is not logged into the selected computer at the time, SetDefault 
 						will Load the selected user's NTUSER.DAT registry Hive and make the selected changes offline.
 				3.2		-Added Current User Display
 	Issues:  	None Discovered as of Yet.
 }	
 #>
 #~~~~~~~~Global \ Script Variables~~~~~~#
 $Script:ping = new-object System.Net.NetworkInformation.Ping
 $Script:tcpClient = New-Object System.Net.Sockets.TcpClient
 $Script:Search = New-Object DirectoryServices.DirectorySearcher([ADSI]â��â��)
 $Script:hku = 2147483651
 $Script:hklm = 2147483650
 $Script:UserPath = "SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList"
 $Script:PrinterPath = "Software\Microsoft\Windows NT\CurrentVersion\Print\Printers"
 $Script:domain = $env:USERDOMAIN
 #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~#
 #~~~~~~~~~~~CUSTOM FUNCTIONS~~~~~~~~~~~~#
 function Test-Alive($comp) {
 	$stat=$ping.Send("$comp")
 	$reply = $stat.status
 	trap {
 		$False
 		continue;
 	}	
     if($reply -eq "Success") {$True}
 }
 function Ping-Port([string]$server, [int]$port) { 
 	trap {  
 		$False 
 		continue; 
 	} 
 	$tcpClient.Connect($server,$port) 
 	if ($tcpClient.Connected) {$True} 
 } 
 Function ADSearch {
 	$Input = $Args[0]
 	$Search.filter = "(&(objectClass=Computer)(Name=$Input))"
 	$Results = $Search.Findall()
 	foreach ($i in $Results)
 	{
 		$Item = $i.properties
 		$lbxServers.Items.Add("$($Item.cn)")
 	}
 }
 function LoadUsers {
 	$UserWMI = [WMIClass]"\\$Computer\root\default:stdRegProv"
 	$Accounts = $UserWMI.EnumKey($hklm,$UserPath)
 	foreach ($Account in $Accounts.snames) {
 			$User = $UserWMI.GetStringValue($hklm,"$UserPath\$Account","ProfileImagePath")
 			$UserName = ($User.svalue).split("\")[2]
 			$cbxUserAccount.Items.Add($Username)
 	}
     $currentuser = (gwmi win32_ComputerSystem -computername $Computer | select UserName).UserName
     $tbxCurrentUser.Text = $currentuser
 
 }
 Function LoadPrinters {
 	$PrinterWMI = [WMIClass]"\\$Computer\root\default:stdRegProv"
 	$Printers = $PrinterWMI.EnumKey($hklm,$PrinterPath)
 	foreach ($Printer in $Printers.snames) {
 			$Print = $PrinterWMI.GetStringValue($hklm,"$PrinterPath\$Printer","Name")
 			$PrinterName = $Print.svalue
 			$cbxPrinter.Items.Add($PrinterName)
 	}
 }
 Function SetWithLocalRegHive {
 #<#
 	$ostest = (gwmi win32_operatingsystem -computername $Computer | select Version | %{$_ -replace "@{Version=", ""} | %{$_ -replace "}", ""})
 	$tbxStatus.ForeColor = [System.Drawing.Color]::FromArgb(255,69,139,0)
 	Switch -wildcard ($ostest) {
         "5.0*" {$tbxStatus.Text = "Windows 2000"; $profileroot = "Documents and Settings"}
         "5.1*" {$tbxStatus.Text = "Windows XP"; $profileroot = "Documents and Settings"}
         "6.0*" {$tbxStatus.Text = "Windows Vista"; $profileroot = "Users" }
         "6.1*" {$tbxStatus.Text = "Windows 7"; $profileroot = "Users"}
     }
 	Reg Load HKLM\TemporaryHive \\$Computer\C$\$profileroot\$UserName\NTUSER.DAT
 	$LocalDefPrinter = (New-Object -ComObject Wscript.shell).RegRead("HKLM\TemporaryHive\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\Device")
     $localDetailsTest = Test-Path "HKLM\TemporaryHive\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Devices"
     If (!$LocalDetailsTest) {
         $tbxStatus.ForeColor = [System.Drawing.Color]::FromArgb(255,255,64,64)
 		$tbxStatus.Text = "Incomplete User Profile"
     }
     Else {
 	    $LocalDefPrinterDetails = (New-Object -ComObject Wscript.Shell).RegRead("HKLM\TemporaryHive\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Devices\$PrinterName")
 	    $LocalDefaultPrinter = ($LocalDefPrinter).split(",")[0]
 	    $Localspool = ($LocalDefPrinterDetails).split(",")[0]
 	    $LocalPort = ($LocalDefPrinterDetails).split(",")[1]
 	    $LocalNewDefPrinter = $PrinterName + "," + $Localspool + "," + $LocalPort            
 	    (New-Object -comobject Wscript.Shell).regWrite("HKLM\TemporaryHive\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\Device", "$LocalNewDefPrinter", "REG_SZ")
         Reg Unload HKLM\TemporaryHive
 	    $tbxStatus.Text = "Default Printer Changed!"
     }	
 }
 Function SetDefaultPrinter { 
     Get-service -ComputerName $Computer -Include 'RemoteRegistry' | Start-Service
 	$ntAccount = new-object System.Security.Principal.NTAccount($domain, $UserName) 
 	$sid = $ntAccount.Translate([System.Security.Principal.SecurityIdentifier]) 
 	$ActionPath = "$sid\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows"
 	$DevicesPath = "$sid\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Devices"
 	$ActionWMI = [WMIClass]"\\$Computer\root\default:stdRegProv"
 	$DefPrinter = $ActionWMI.GetStringValue($hku,"$ActionPath","Device")
     $DefPrinterDetails = $ActionWMI.GetStringValue($hku,"$DevicesPath","$PrinterName")
 	$testRegKey = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey($hku,$Computer)
 	$testReg = $testRegKey.OpenSubKey("$sid")
 		If (!$testReg) {
             SetWithLocalRegHive
 		}
 		Else {
 			$DefaultPrinter = ($DefPrinter.svalue).split(",")[0]
 			$spool = ($DefPrinterDetails.svalue).split(",")[0]
 			$Port = ($DefPrinterDetails.svalue).split(",")[1]
 			$NewDefPrinter = $PrinterName + "," + $spool + "," + $Port
 			
 			$tbxStatus.ForeColor = [System.Drawing.Color]::FromArgb(255,69,139,0)
 			$tbxStatus.Text = "Default Printer Changed!"
 		}
 		$cbxUserAccount.ResetText()
 		$cbxPrinter.ResetText()
 }
 #~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~#
 function GenerateForm {
 ########################################################################
 ########################################################################
 
 [reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 [reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
 [reflection.assembly]::loadwithpartialname("System.Windows.Forms.VisualStyles") | Out-Null
 [System.Windows.Forms.Application]::EnableVisualStyles()
 
 $FrmMain = New-Object System.Windows.Forms.Form
 $pgbProgress = New-Object System.Windows.Forms.ProgressBar
 $lblPrinter = New-Object System.Windows.Forms.Label
 $lblUserAccount = New-Object System.Windows.Forms.Label
 $lblSearch = New-Object System.Windows.Forms.Label
 $btnSearch = New-Object System.Windows.Forms.Button
 $tbxSearch = New-Object System.Windows.Forms.TextBox
 $btnClose = New-Object System.Windows.Forms.Button
 $btnLoad = New-Object System.Windows.Forms.Button
 $tbxServer = New-Object System.Windows.Forms.TextBox
 $tbxStatus = New-Object System.Windows.Forms.TextBox
 $tbxCurrentUser = New-Object System.Windows.Forms.TextBox
 $lbxServers = New-Object System.Windows.Forms.ListBox
 $btnSetDefault = New-Object System.Windows.Forms.Button
 $cbxPrinter = New-Object System.Windows.Forms.ComboBox
 $cbxUserAccount = New-Object System.Windows.Forms.ComboBox
 $InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
 #----------------------------------------------
 #----------------------------------------------
 $btnClose_OnClick= 
 {
 	$FrmMain.Close()
 }
 
 $lbxServers.add_SelectedIndexChanged({ 
    	$tbxStatus.Text = $Null
 	$tbxStatus.Refresh()
 	$tbxServer.Text = $lbxServers.SelectedItem
 	$tbxServer.Refresh()
     $tbxCurrentUser.Text = $null
     $tbxCurrentUser.Refresh()
     $cbxPrinter.Items.clear()
     $cbxPrinter.Text = $null
     $cbxPrinter.Refresh()
     $cbxUserAccount.Items.clear()
     $cbxUserAccount.Text = $null
     $cbxUserAccount.Refresh()
 
 }) 
 
 $btnLoad_OnClick= 
 {
 	$tbxStatus.ForeColor = [System.Drawing.Color]::FromArgb(255,255,255,0)
 	$tbxStatus.Text = "Loading..."
 	$tbxStatus.Refresh()
 	$Computer = $tbxServer.Text
 	If (Test-Alive $Computer) {	
 		if ( (ping-port $Computer 135) -and (ping-port $Computer 445) ) {
 			LoadUsers $Computer
 			LoadPrinters $Computer
 			$tbxStatus.ForeColor = [System.Drawing.Color]::FromArgb(255,69,139,0)
 			$tbxStatus.Text = "Ready..."
 		}
 		Else { 
 			$tbxStatus.ForeColor = [System.Drawing.Color]::FromArgb(255,255,64,64)
 			$tbxStatus.Text = "RPC Server Not Available" 
 			}
 	}
 	Else { 
 		$tbxStatus.ForeColor = [System.Drawing.Color]::FromArgb(255,255,64,64)
 		$tbxStatus.Text = "Server Offline" 
 		}
 	$pgbProgress.Visible = $False
 }
 
 $btnSetDefault_OnClick= 
 {
     $Script:Computer = $tbxServer.Text 
 	$Script:UserName = $cbxUserAccount.SelectedItem 
 	$Script:PrinterName = $cbxPrinter.SelectedItem
 	SetDefaultPrinter
 }
 
 $btnSearch_OnClick= 
 {
 	$lbxServers.Items.Clear()
 	$Criteria = $tbxSearch.Text
 	ADSearch $Criteria
 }
 
 $OnLoadForm_StateCorrection=
 	$FrmMain.WindowState = $InitialFormWindowState
 }
 
 #----------------------------------------------
 $FrmMain.CancelButton = $btnClose
 $FrmMain.BackColor = [System.Drawing.Color]::FromArgb(255,166,202,240)
 $FrmMain.Text = "Set Default Printer"
 $FrmMain.Name = "FrmMain"
 $FrmMain.KeyPreview = $True
 $FrmMain.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 439
 $System_Drawing_Size.Height = 272
 $FrmMain.ClientSize = $System_Drawing_Size
 $FrmMain.AcceptButton = $btnSearch
 $FrmMain.FormBorderStyle = 2
 
 $lblSearch.TabIndex = 0
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 100
 $System_Drawing_Size.Height = 14
 $lblSearch.Size = $System_Drawing_Size
 $lblSearch.Text = "Search"
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 9
 $lblSearch.Location = $System_Drawing_Point
 $lblSearch.DataBindings.DefaultDataSourceUpdateMode = 0
 $lblSearch.Name = "lblSearch"
 $FrmMain.Controls.Add($lblSearch)
 
 $lblPrinter.TabIndex = 0
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 100
 $System_Drawing_Size.Height = 12
 $lblPrinter.Size = $System_Drawing_Size
 $lblPrinter.Text = "Printer:"
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 258
 $System_Drawing_Point.Y = 185
 $lblPrinter.Location = $System_Drawing_Point
 $lblPrinter.DataBindings.DefaultDataSourceUpdateMode = 0
 $lblPrinter.Name = "lblPrinter"
 $FrmMain.Controls.Add($lblPrinter)
 
 $lblUserAccount.TabIndex = 0
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 100
 $System_Drawing_Size.Height = 13
 $lblUserAccount.Size = $System_Drawing_Size
 $lblUserAccount.Text = "User Account:"
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 259
 $System_Drawing_Point.Y = 145
 $lblUserAccount.Location = $System_Drawing_Point
 $lblUserAccount.DataBindings.DefaultDataSourceUpdateMode = 0
 $lblUserAccount.Name = "lblUserAccount"
 $FrmMain.Controls.Add($lblUserAccount)
 
 $pgbProgress.DataBindings.DefaultDataSourceUpdateMode = 0
 $pgbProgress.BackColor = [System.Drawing.Color]::FromArgb(255,166,202,240)
 $pgbProgress.ForeColor = [System.Drawing.Color]::FromArgb(255,0,255,0)
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 191
 $System_Drawing_Size.Height = 8
 $pgbProgress.Size = $System_Drawing_Size
 $pgbProgress.Step = 1
 $pgbProgress.TabIndex = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 246
 $System_Drawing_Point.Y = 105
 $pgbProgress.Location = $System_Drawing_Point
 $pgbProgress.MarqueeAnimationSpeed = 50
 $pgbProgress.Visible = $False
 $pgbProgress.Name = "pgbProgress"
 $FrmMain.Controls.Add($pgbProgress)
 
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 168
 $System_Drawing_Size.Height = 17
 $tbxStatus.Size = $System_Drawing_Size
 $tbxStatus.DataBindings.DefaultDataSourceUpdateMode = 0
 $tbxStatus.TextAlign = 2
 $tbxStatus.BorderStyle = 0
 $tbxStatus.Font = New-Object System.Drawing.Font("Microsoft Sans Serif",10,1,3,0)
 $tbxStatus.Name = "tbxStatus"
 $tbxStatus.BackColor = [System.Drawing.Color]::FromArgb(255,166,202,240)
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 259
 $System_Drawing_Point.Y = 88
 $tbxStatus.Location = $System_Drawing_Point
 $tbxStatus.TabIndex = 20
 $FrmMain.Controls.Add($tbxStatus)
 
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 168
 $System_Drawing_Size.Height = 17
 $tbxServer.Size = $System_Drawing_Size
 $tbxServer.DataBindings.DefaultDataSourceUpdateMode = 0
 $tbxServer.TextAlign = 2
 $tbxServer.BorderStyle = 0
 $tbxServer.Font = New-Object System.Drawing.Font("Microsoft Sans Serif",10,1,3,1)
 $tbxServer.Name = "tbxServer"
 $tbxServer.BackColor = [System.Drawing.Color]::FromArgb(255,166,202,240)
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 259
 $System_Drawing_Point.Y = 32
 $tbxServer.Location = $System_Drawing_Point
 $tbxServer.TabIndex = 19
 $FrmMain.Controls.Add($tbxServer)
 
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 168
 $System_Drawing_Size.Height = 17
 $tbxCurrentUser.Size = $System_Drawing_Size
 $tbxCurrentUser.DataBindings.DefaultDataSourceUpdateMode = 0
 $tbxCurrentUser.TextAlign = 2
 $tbxCurrentUser.BorderStyle = 0
 $tbxCurrentUser.Name = "tbxCurrentUser"
 $tbxCurrentUser.BackColor = [System.Drawing.Color]::FromArgb(255,166,202,240)
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 259
 $System_Drawing_Point.Y = 60
 $tbxCurrentUser.Location = $System_Drawing_Point
 $tbxCurrentUser.TabIndex = 20
 $FrmMain.Controls.Add($tbxCurrentUser)
 
 $btnSearch.TabIndex = 2
 $btnSearch.Name = "btnSearch"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 60
 $System_Drawing_Size.Height = 23
 $btnSearch.Size = $System_Drawing_Size
 $btnSearch.UseVisualStyleBackColor = $True
 $btnSearch.Text = "Search"
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 180
 $System_Drawing_Point.Y = 26
 $btnSearch.Location = $System_Drawing_Point
 $btnSearch.DataBindings.DefaultDataSourceUpdateMode = 0
 $btnSearch.add_Click($btnSearch_OnClick)
 $FrmMain.Controls.Add($btnSearch)
 
 $btnLoad.TabIndex = 4
 $btnLoad.Name = "btnLoad"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 60
 $System_Drawing_Size.Height = 23
 $btnLoad.Size = $System_Drawing_Size
 $btnLoad.UseVisualStyleBackColor = $True
 $btnLoad.Text = "Load"
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 310
 $System_Drawing_Point.Y = 120
 $btnLoad.Location = $System_Drawing_Point
 $btnLoad.DataBindings.DefaultDataSourceUpdateMode = 0
 $btnLoad.add_Click($btnLoad_OnClick)
 $FrmMain.Controls.Add($btnLoad)
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 351
 $System_Drawing_Point.Y = 236
 $btnClose.Location = $System_Drawing_Point
 $btnClose.DataBindings.DefaultDataSourceUpdateMode = 0
 $btnClose.DialogResult = 2
 $btnClose.add_Click($btnClose_OnClick)
 $FrmMain.Controls.Add($btnClose)
 
 $btnSetDefault.TabIndex = 7
 $btnSetDefault.Name = "btnSetDefault"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 75
 $System_Drawing_Size.Height = 23
 $btnSetDefault.Size = $System_Drawing_Size
 $btnSetDefault.UseVisualStyleBackColor = $True
 $btnSetDefault.Text = "Set Default"
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 259
 $System_Drawing_Point.Y = 236
 $btnSetDefault.Location = $System_Drawing_Point
 $btnSetDefault.DataBindings.DefaultDataSourceUpdateMode = 0
 $btnSetDefault.add_Click($btnSetDefault_OnClick)
 $FrmMain.Controls.Add($btnSetDefault)
 
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 161
 $System_Drawing_Size.Height = 20
 $tbxSearch.Size = $System_Drawing_Size
 $tbxSearch.DataBindings.DefaultDataSourceUpdateMode = 0
 $tbxSearch.Name = "tbxSearch"
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 29
 $tbxSearch.Location = $System_Drawing_Point
 $tbxSearch.TabIndex = 1
 $FrmMain.Controls.Add($tbxSearch)
 $btnClose.TabIndex = 8
 $btnClose.Name = "btnClose"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 75
 $System_Drawing_Size.Height = 23
 $btnClose.Size = $System_Drawing_Size
 $btnClose.UseVisualStyleBackColor = $True
 $btnClose.Text = "Close"
 
 #$lbxServers.UseCompatibleStateImageBehavior = $False
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 227
 $System_Drawing_Size.Height = 205
 $lbxServers.Size = $System_Drawing_Size
 $lbxServers.DataBindings.DefaultDataSourceUpdateMode = 0
 $lbxServers.Name = "lbxServers"
 #$lbxServers.View = 3
 $lbxServers.TabIndex = 3
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 55
 $lbxServers.Location = $System_Drawing_Point
 $FrmMain.Controls.Add($lbxServers)
 
 $cbxPrinter.FormattingEnabled = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 168
 $System_Drawing_Size.Height = 21
 $cbxPrinter.Size = $System_Drawing_Size
 $cbxPrinter.DataBindings.DefaultDataSourceUpdateMode = 0
 $cbxPrinter.Name = "cbxPrinter"
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 258
 $System_Drawing_Point.Y = 200
 $cbxPrinter.Location = $System_Drawing_Point
 $cbxPrinter.TabIndex = 6
 $FrmMain.Controls.Add($cbxPrinter)
 
 $cbxUserAccount.FormattingEnabled = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Width = 168
 $System_Drawing_Size.Height = 21
 $cbxUserAccount.Size = $System_Drawing_Size
 $cbxUserAccount.DataBindings.DefaultDataSourceUpdateMode = 0
 $cbxUserAccount.Name = "cbxUserAccount"
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 258
 $System_Drawing_Point.Y = 161
 $cbxUserAccount.Location = $System_Drawing_Point
 $cbxUserAccount.TabIndex = 5
 $FrmMain.Controls.Add($cbxUserAccount)
 
 
 $InitialFormWindowState = $FrmMain.WindowState
 $FrmMain.add_Load($OnLoadForm_StateCorrection)
 $FrmMain.ShowDialog()| Out-Null
 
 
 GenerateForm
`

