---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6266
Published Date: 2016-03-21t19
Archived Date: 2016-10-07t11
---

# ise nativeconsole hooks - 

## Description

a demo for some fun native-console hooking in powershell ise

## Comments



## Usage



## TODO



## function

`receive-output`

## Code

`#
 #
 add-type -Path ~\Projects\PoshConsole\Huddled\Interop\NativeConsole.cs
 
 $NativeConsole = New-Object Huddled.Interop.NativeConsole
 
 $ConsoleError = Register-ObjectEvent -InputObject $NativeConsole -EventName WriteError -Action { 
     Add-Content -Path $Pwd\Error.log -Value $EventArgs.Text
     Add-Content -Path $Pwd\Output.log -Value $EventArgs.Text
     Write-Error $EventArgs.Text
     Write-Host $EventArgs.Text -ForegroundColor Red
 }
 
 $ConsoleOutput = Register-ObjectEvent -InputObject $NativeConsole -EventName WriteOutput -Action {
     Add-Content -Path $Pwd\Output.log -Value $EventArgs.Text
     Write-Output $EventArgs.Text 
     Write-Host $EventArgs.Text
 }
 
 cmd /c "dir && dir brokenthing"
 
 function receive-output {
     Receive-Job $ConsoleOutput, $ConsoleError
 }
`

