---
Author: glnsize
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 901
Published Date: 
Archived Date: 2009-03-03t09
---

# disconnect-vmhost - 

## Description

disconnect a vmhost from vcenter

## Comments



## Usage



## TODO



## function

`disconnect-vmhost`

## Code

`  Example using the VI API real world use Set-VMHost -State
 #
 #
 Function Disconnect-VMHost {
     <#
     .Summary
         Used to Disconnect a Connected host from vCenter.
     .Parameter VMHost
         VMHost to Disconnect to virtual center
     .Example
         Get-VMHost | Where-Object {$_.state -eq "Connected"} | Disconnect-VMHost
         
         Will Attempt to Disconnect any host that are currently Connected.
     .Example
         Disconnect-VMHost -Name ESX1.get-admin.local
         
         Will Disconnect ESX1 From vCenter
     #>
     [CmdletBinding(
         SupportsShouldProcess=$True,
 	    SupportsTransactions=$False,
 	    ConfirmImpact="low",
 	    DefaultParameterSetName="ByString"
 	)]
     Param(
         [Parameter(
             Mandatory=$True,
             Valuefrompipeline=$true,
             ParameterSetName="ByObj"
         )]
         [VMware.VimAutomation.Client20.VMHostImpl[]]
         $VMHost,
         
         [Parameter(
             Mandatory=$True,
             Position=0,
             ParameterSetName="ByString"
         )]
         [string[]]
         $Name
     )
     Begin {
         IF ($Name) {
             $VMHost = $Name|%{ Get-VMHost -Name $_ }
         }
     }
     process {
         Foreach ($VMHostImpl in ($VMHost|Get-View)) {
             if ($pscmdlet.ShouldProcess($VMHostImpl.name)) {
                 $VMHostImpl.DisconnectHost_Task()
             }
         }
     }
 }
`

