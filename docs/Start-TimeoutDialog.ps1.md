---
Author: grant carthew
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2610
Published Date: 2011-04-07t18
Archived Date: 2011-04-10t07
---

# start-timeoutdialog - 

## Description

displays a custom dialog box for a timeout period. the dialog box contains a message, two buttons and a countdown timer. the button text and other options are set through the parameters. the wshshell.popup has a bug so replace it with this script.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
     .Synopsis
      Displays a message box that will timeout after a set number of seconds.
      This script was written because WSHShell.Popup has a bug.
      
      Platform: Windows 7 32bit, Windows 2008 R2 64bit
      Author: Grant Carthew.
      Date: Apr 2011
      Updated: 
 
     .Description
      When calling this script it creates a message box with custom settings
      and displays the message.   There is a countdown timer displayed on the
      message box to inform the user when the message will timeout.
      
      If the user presses Button1 then the text on Button1 is returned.
      If the user presses Button2 then the text on Button2 is returned.
      If the message box times out then the string TIMEOUT is returned.
      
     .Parameter Title
      The title of the message box.
      Default = "Timeout"
      
     .Parameter Message
      The message to display on the message box.
      Default = "Timeout Message"
      
     .Parameter Button1Text
      The text to display on the left hand button.
      This text is returned if the user clicks the button.
 	 Button1 is the default accept button so ENTER will press Button1
      Default = "OK"
      
     .Parameter Button2Text
      The text to display on the right hand button.
      This text is returned if the user clicks the button.
 	 Button2 is the default cancel button so ESC will press Button2
      Default = "Cancel"
      
     .Parameter Seconds
      The timeout value for the message box.
      If the message box is displayed for this amount of time then
      it will close and the string TIMEOUT will be returned.
      Default = 30
      
     .Example
      Start-TimeoutDialog -Title "Shutdown" -Message "This system will shutdown in 50 minutes." -Seconds 3000
 #> 
 
 param
 (
     [parameter(Mandatory=$false)]
     [String]
     $Title = "Timeout",
     
     [parameter(Mandatory=$false)]
     [String]
     $Message = "Timeout Message",
     
     [parameter(Mandatory=$false)]
     [String]
     $Button1Text = "OK",
     
     [parameter(Mandatory=$false)]
     [String]
     $Button2Text = "Cancel",
     
     [parameter(Mandatory=$false)]
     [Int]
     $Seconds = 30
 )
 
 BEGIN
 {
     Write-Verbose -Message ("Begin: " + $MyInvocation.MyCommand.Path)
     [System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") | Out-Null
     $F = $null
     $B1 = $null
     $B2 = $null
     $L = $null
     $TB = $null
     $Timer = $null
     $Result = $null
     $TimeLeft = New-TimeSpan -Seconds $Seconds
     $ONESEC = New-TimeSpan -Seconds 1
 }
 
 PROCESS
 {
     $F = New-Object -TypeName System.Windows.Forms.Form 
     $F.Text = $Title
     $F.Size = New-Object -TypeName System.Drawing.Size(300,150) 
     $F.StartPosition = "CenterScreen"
     $F.Topmost = $true
     $F.KeyPreview = $true
     $F.ShowInTaskbar = $false
     $F.ShowIcon = $false
     $F.FormBorderStyle = "FixedDialog"
     $F.MaximizeBox = $false
     $F.MinimizeBox = $false
 
     $B1 = New-Object -TypeName System.Windows.Forms.Button
     $B1.Size = New-Object -TypeName System.Drawing.Size(75,23)
     $B1.Location = New-Object -TypeName System.Drawing.Size(125,90)
     $B1.Text = $Button1Text
     $B1.Add_Click({$Result=$Button1Text;$F.Close()})
     $F.Controls.Add($B1)
     $F.AcceptButton = $B1
 
     $B2 = New-Object -TypeName System.Windows.Forms.Button
     $B2.Size = New-Object -TypeName System.Drawing.Size(75,23)
     $B2.Location = New-Object -TypeName System.Drawing.Size(210,90)
     $B2.Text = $Button2Text
     $B2.Add_Click({$Result=$Button2Text;$F.Close()})
     $F.Controls.Add($B2)
     $F.CancelButton = $B2
 
     $L = New-Object -TypeName System.Windows.Forms.Label
     $L.Size = New-Object -TypeName System.Drawing.Size(280,65) 
     $L.Location = New-Object -TypeName System.Drawing.Size(10,15) 
     $L.Text = $Message
     $F.Controls.Add($L) 
 
     $TB = New-Object -TypeName System.Windows.Forms.TextBox 
     $TB.Size = New-Object -TypeName System.Drawing.Size(100,20) 
     $TB.Location = New-Object -TypeName System.Drawing.Size(10,91)
     $TB.TextAlign = "Center"
     $TB.ReadOnly = $true
     $TB.Text = [String]$TimeLeft.Hours + " : " + $TimeLeft.Minutes + " : " + $TimeLeft.Seconds
     $F.Controls.Add($TB)
     
     $Timer = New-Object -TypeName System.Windows.Forms.Timer
     $Timer.Interval = 1000
     $Timer.Add_Tick({$TimeLeft = $TimeLeft - $ONESEC;$TB.Text = [String]$TimeLeft.Hours + " : " + $TimeLeft.Minutes + " : " + $TimeLeft.Seconds;if ($TimeLeft.TotalSeconds -lt 1) { $Result = "TIMEOUT"; $F.Close() } })
     $Timer.Start()
 
     $F.Add_Shown({$F.Activate()})
     $F.ShowDialog() | Out-Null
     $Result
 }
 
 END
 {
     Write-Verbose -Message ("End: " + $MyInvocation.MyCommand.Name)
 }
`

