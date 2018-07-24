---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1582
Published Date: 2010-01-16t11
Archived Date: 2014-05-25t04
---

# stop-pipeline - 

## Description

use this to stop scripts running in powershell.exe in mid-stride.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $Wasp = Add-Type -MemberDefinition @'
 [DllImport("user32.dll")]
 [return: MarshalAs(UnmanagedType.Bool)]
 public static extern bool SetForegroundWindow(IntPtr hWnd);
 '@ -Passthru
 
 $null = Add-Type -Assembly System.Windows.Forms
 
 filter Stop-Pipeline(  [scriptblock]$condition = {$true} ) {
    if (& $condition) {
       if($Wasp::SetForegroundWindow( (Get-Process -id $PID).MainWindowHandle )) {
          [System.Windows.Forms.SendKeys]::SendWait("^C")
       } else {
          Throw (New-Object System.Management.Automation.PipelineStoppedException)
       }
       $_
    }
 }
`

