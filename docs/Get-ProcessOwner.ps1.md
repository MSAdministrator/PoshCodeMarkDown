---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4758
Published Date: 2014-01-02t07
Archived Date: 2014-01-04t21
---

# get-processowner - 

## Description

i’m continuing to experiment with powershell

## Comments



## Usage



## TODO



## function

`get-processowner`

## Code

`#
 #
 function Get-ProcessOwner {
   <#
     .EXAMPLE
         PS C:\>Get-ProcessOwner (ps notepad)
     .EXAMPLE
         PS C:\>ps notepad | Get-ProcessOwner
     .NOTES
         Author: greg zakharov
   #>
   [CmdletBinding(SupportsShouldProcess=$true)]
   param(
     [Parameter(Mandatory=$true, Position=0, ValueFromPipeline=$true)]
     [Diagnostics.Process[]]$Processes
   )
   
   begin {
     $StopProcessCommand = ([AppDomain]::CurrentDomain.GetAssemblies() | ? {
       $_.Location.Split('\')[-1].Equals('Microsoft.PowerShell.Commands.Management.dll')
     }).GetType('Microsoft.PowerShell.Commands.StopProcessCommand')
     
     $GetProcessOwnerId = $StopProcessCommand.GetMethod(
       'GetProcessOwnerId', [Reflection.BindingFlags]'NonPublic, Instance'
     )
     
     $type = New-Object $StopProcessCommand
   }
   process {
     $Processes | % {
       if ($PSCmdlet.ShouldProcess(('{0} PID:{1}' -f $_.ProcessName, $_.Id), 'Check owner of process')) {
         try {
           '{0} {1} {2}' -f $_.ProcessName, $_.Id, $GetProcessOwnerId.Invoke($type, $_)
         }
         catch {
           $_.Exception.Message.Split(':')[1].Trim() -replace '\"', ''
         }
       }
     }
   }
   end {
   }
 }
`

