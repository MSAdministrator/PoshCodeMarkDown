---
Author: bryan jaudon
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3735
Published Date: 2012-11-01t09
Archived Date: 2015-12-26t20
---

# numlock notifier - 

## Description

displays numlock status icon in the taskbar notification area. shows balloon tip if status has been changed.

## Comments



## Usage



## TODO



## script

`hide-powershell`

## Code

`#
 #
 <#
 
     .NOTES
     Name     : NumLockNotifier.ps1
     Author   : Bryan Jaudon <bryan.jaudon@gmail.com>
     Version  : 1.0
     Date     : 10/25/2012
 
     .Description
     Adds a notification icon to show current NumLock status. Double clicking or by using the context menu, allows for 
     toggling of the NumLock status.
 
 
 #>
 
 
 param([switch]$Debug)
 
 function CheckNumLock {
     $NumLockStatus=[console]::NumberLock
     Write-Debug "CheckNumLock - Previous Reading: $Script:PreviousNumLock - Current Reading: $NumLockStatus"
     if ($PreviousNumLock -ne $NumLockStatus) {
         if ($NumLockStatus -eq $True) {
             Write-Debug "CheckNumLock - Update NumLock NotificationIcon to ON" 
             $NotifyIcon.Icon = $IconOn 
             $NotifyIcon.BalloonTipTitle = "NumLock is On"
             $NotifyIcon.Text="NumLock Status - On"
             $NotifyIcon.BalloonTipText = "NumLock status has changed to On." 
         }
         else { 
             Write-Debug "CheckNumLock - Update NumLock NotificationIcon to OFF"
             $NotifyIcon.Icon = $IconOff 
             $NotifyIcon.BalloonTipTitle = "NumLock is Off"
             $NotifyIcon.Text="NumLock Status - Off"
             $NotifyIcon.BalloonTipText = "NumLock status has changed to Off." 
         }
         Write-Debug "CheckNumLock - Show NotificationIcon BaloonTip"
         $NotifyIcon.BalloonTipIcon = "Info" 
         $NotifyIcon.Visible = $True 
         $NotifyIcon.ShowBalloonTip(50000)  
     }
     $Script:PreviousNumLock=$NumLockStatus
 }
 
 function ToggleNumLock {
     $wshObject=New-Object -ComObject "WScript.Shell"
     $wshObject.SendKeys('{NUMLOCK}')
     Write-Debug "ToggleNumLock - wshObject Sendkeys(NUMLOCK)"
 }
 
 Function Hide-PowerShell { $null = $showWindowAsync::ShowWindowAsync((Get-Process -PID $pid).MainWindowHandle, 0) }
 
 [void][System.Reflection.Assembly]::LoadWithPartialName("System.windows.forms")
 [void][reflection.assembly]::loadwithpartialname("System.Drawing")
 
 [string]$IconOnb64=@"
 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
 "@
 
 $IconOnstream=[System.IO.MemoryStream][System.Convert]::FromBase64String($IconOnb64)
 $IconOnbmp=[System.Drawing.Bitmap][System.Drawing.Image]::FromStream($IconOnstream)
 $IconOnhandle=$IconOnbmp.GetHicon()
 $IconOn=[System.Drawing.Icon]::FromHandle($IconOnhandle)
 
 [string]$IconOffb64=@"
 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
 "@
 
 $IconOffstream=[System.IO.MemoryStream][System.Convert]::FromBase64String($IconOffb64)
 $IconOffbmp=[System.Drawing.Bitmap][System.Drawing.Image]::FromStream($IconOffstream)
 $IconOffhandle=$IconOffbmp.GetHicon()
 $IconOff=[System.Drawing.Icon]::FromHandle($IconOffhandle)
 
 $form1 = New-Object System.Windows.Forms.form
 $NotifyIcon= New-Object System.Windows.Forms.NotifyIcon
 $ContextMenu = New-Object System.Windows.Forms.ContextMenu
 $MenuItemToggle = New-Object System.Windows.Forms.MenuItem
 $MenuItemExit = New-Object System.Windows.Forms.MenuItem
 $Timer = New-Object System.Windows.Forms.Timer
 
 $form1.ShowInTaskbar = $false
 $form1.WindowState = "minimized"
 
 $NotifyIcon.ContextMenu = $ContextMenu
 $NotifyIcon.contextMenu.MenuItems.AddRange($MenuItemToggle)
 $NotifyIcon.contextMenu.MenuItems.AddRange($MenuItemExit)
 $NotifyIcon.Visible = $True
 $NotifyIcon.add_DoubleClick({ Write-Debug "NotifyIcon - DoubleClickEvent: ToggleNumLock";ToggleNumLock})
 
 $Timer.Interval =  1000
 $Timer.add_Tick({ CheckNumLock })
 $Timer.start()
 
 $MenuItemToggle.Index = 0
 $MenuItemToggle.Text = "&Toggle NumLock"
 $MenuItemToggle.add_Click({ Write-Debug "NotifyIcon - MenuItemEvent: ToggleNumLock"; ToggleNumLock })
 
 $MenuItemExit.Index = 1
 $MenuItemExit.Text = "E&xit"
 $MenuItemExit.add_Click({ Write-Debug "NotifyIcon - ExitEvent"; $Timer.stop(); $NotifyIcon.Visible = $False; $form1.close() })
 
 $script:showWindowAsync = Add-Type -MemberDefinition @"
 [DllImport("user32.dll")]
 public static extern bool ShowWindowAsync(IntPtr hWnd, int nCmdShow);
 "@ -name Win32ShowWindowAsync -namespace Win32Functions -PassThru
 
 if ($host.name -ne "Windows PowerShell ISE Host") { 
     if ($Debug -ne $True) { Hide-PowerShell }
     else { $DebugPreference="Continue" }
 }
 
 $Script:PreviousNumLock=[console]::NumberLock
 if ($Script:PreviousNumLock -eq $True) {
     Write-Debug "Initilization - NumLock On"
     $NotifyIcon.Icon = $IconOn 
     $NotifyIcon.Text="NumLock Status - On"
 }
 else {
     Write-Debug "Initilization - NumLock Off"
     $NotifyIcon.Icon = $IconOff 
     $NotifyIcon.Text="NumLock Status - Off"
 }
 
 [void][System.Windows.Forms.Application]::Run($form1)
`

