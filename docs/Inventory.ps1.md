---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 6.0.6002
Encoding: ascii
License: cc0
PoshCode ID: 2172
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t00
---

# inventory.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

`get-inventory`

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Serves as the configuration script for a custom remoting endpoint that
 exposes only the Get-Inventory custom command.
 
 .EXAMPLE
 
 PS >Register-PsSessionConfiguration Inventory `
     -StartupScript 'C:\Program Files\Endpoints\Inventory.ps1'
 PS >Enter-PsSession leeholmes1c23 -ConfigurationName Inventory
 
 [leeholmes1c23]: [Inventory] > Get-Command
 
 CommandType     Name                          Definition
 -----------     ----                          ----------
 Function        Exit-PSSession                [CmdletBinding()]...
 Function        Get-Command                   [CmdletBinding()]...
 Function        Get-FormatData                [CmdletBinding()]...
 Function        Get-Help                      [CmdletBinding()]...
 Function        Get-Inventory                 ...
 Function        Measure-Object                [CmdletBinding()]...
 Function        Out-Default                   [CmdletBinding()]...
 Function        prompt                        ...
 Function        Select-Object                 [CmdletBinding()]...
 
 [leeholmes1c23]: [Inventory] > Get-Inventory
 
 SystemDirectory : C:\Windows\system32
 Organization    :
 BuildNumber     : 6002
 RegisteredUser  : Lee Holmes
 SerialNumber    : 89580-433-1295803-71477
 Version         : 6.0.6002
 
 [leeholmes1c23]: [Inventory] > 1+1
 The syntax is not supported by this runspace. This might be because it is
 in no-language mode.
     + CategoryInfo          :
     + FullyQualifiedErrorId : ScriptsNotAllowed
 
 [leeholmes1c23]: [Inventory] > Exit-PsSession
 PS >
 
 #>
 
 Set-StrictMode -Off
 
 function Get-Inventory
 {
     Get-WmiObject Win32_OperatingSystem
 }
 
 function Prompt
 {
     "[Inventory] > "
 }
 
 $exportedCommands = "Get-Inventory","Prompt"
 
 $issType = [System.Management.Automation.Runspaces.InitialSessionState]
 $iss = $issType::CreateRestricted("RemoteServer")
 
 $issHashtable = @{}
 foreach($command in $iss.Commands)
 {
     $issHashtable[$command.Name + "-" + $command.CommandType] = $command
 }
 
 foreach($function in $iss.Commands |
     Where-Object { $_.CommandType -eq "Function" })
 {
     Set-Content "function:\$($function.Name)" -Value $function.Definition
 }
 
 foreach($command in Get-Command)
 {
     if($exportedCommands -contains $command.Name) { continue }
 
     $issCommand = $issHashtable[$command.Name + "-" + $command.CommandType]
     if((-not $issCommand) -or ($issCommand.Visibility -ne "Public"))
     {
         $command.Visibility = "Private"
     }
 }
 
 $executionContext.SessionState.Scripts.Clear()
 $executionContext.SessionState.Applications.Clear()
 $executionContext.SessionState.LanguageMode = "NoLanguage"
`

