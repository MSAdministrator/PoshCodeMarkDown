---
Author: jeff patton
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2718
Published Date: 2013-06-08t08
Archived Date: 2013-09-23t23
---

# get-vmhostnetworks - 

## Description

after connecting to your vi server, we get a list of virtual switches on the datacenter and from that we pull out the vhostid that matches the server we passed in at the command-line. using the vhostid we return a list of networks objects on that server.

## Comments



## Usage



## TODO



## function

`get-vmhostnetworks`

## Code

`#
 #
 Function Get-VMHostNetworks
 {
     <#
         .SYNOPSIS
             Return a list of networks from a given host
         .DESCRIPTION
             After connecting to your VI server, we get a list of virtual switches on the datacenter and from
             that we pull out the VHostID that matches the server we passed in at the command-line. Using the
             VHostID we return a list of networks objects on that server.
         .PARAMETER VMHost
             The name of the VMWare Host Server to pull networks from
         .PARAMETER VIServer
             The name of your VSPhere Server
         .EXAMPLE
             Get-VMHostNetworks -VMHost v1.copmany.com -VIServer vc.company.com
             Name                      Key                            VLanId PortBinding NumPorts
             ----                      ---                            ------ ----------- --------
             Management Network        key-vim.host.PortGroup-Mana... 0
             DMZ Network               key-vim.host.PortGroup-DMZ ... 100
             Admin Network             key-vim.host.PortGroup-Admi... 101
 
             Description
             -----------
             This shows the output from the command using all parameters.
         .NOTES
             This script requires the VMware vSphere PowerCLI to be downloaded and installed, please see
             the second link for the download.
         .LINK
         .LINK
             http://communities.vmware.com/community/vmtn/server/vsphere/automationtools/powercli
         .LINK
             http://www.vmware.com/support/developer/PowerCLI/PowerCLI41U1/html/Connect-VIServer.html
         .LINK
             http://www.vmware.com/support/developer/PowerCLI/PowerCLI41U1/html/Get-VirtualSwitch.html
         .LINK
             http://www.vmware.com/support/developer/PowerCLI/PowerCLI41U1/html/Get-VirtualPortGroup.html
     #>
     
     Param
     (
         [string]$VMHost,
         [string]$VIServer
     )
     
     Begin
     {
         Try
         {
             If ($DefaultVIServers -eq $null)
             {
                 Connect-VIServer -Server $VIServer |Out-Null
                 }
             $VSwitches = Get-VirtualSwitch
             }
         Catch
         {
             Return $Error[0]
             }
         }
 
     Process
     {
         foreach ($Vswitch in $VSwitches)
         {
             If ($VSwitch.VMHost.Name -like "$($VMhost)*")
             {
                 $VHostID = $VSwitch.VMHost.Id
                 }
             }
 
         $VMNetworks = Get-VirtualPortGroup |Where-Object {$_.VMhostID -eq $VhostID}
         }
 
     End
     {
         Return $VMNetworks
         }
 }
`

