---
Author: jgrote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3292
Published Date: 2012-03-17t11
Archived Date: 2012-03-28t09
---

# set-esxremotecli - 

## Description

this script enables or disables the remote cli services on a host. it accepts multiple hosts through the pipeline, useful for enabling or disabling ssh on several hosts at once.

## Comments



## Usage



## TODO



## function

`set-esxremotecli`

## Code

`#
 #
 
 function Set-ESXRemoteCLI() {
     Param([parameter(Mandatory=$true,ValueFromPipeline=$true)]$VMHost,
         [parameter(Mandatory=$true)][Boolean]$enabled,
         [switch]$onboot
     )
     
     Process {
         $VMHost | foreach {
             write-progress -id 1 -activity "Modifying Remote CLI on $VMHost" -status "Accessing Remote CLI services on $VMHost"
             $CLIservice = get-vmhostservice $_ | where {$_.key -match "TSM*"}
             if (!$CLIservice) {write-error "No Remote CLI Services found on this server. Please ensure it is vSphere 4 or later."}
             
             $CLIservice | foreach {
                 $serviceToStart = $_
                 if ($enabled) {
                     write-progress -id 1 -activity "Modifying Remote CLI on $VMHost" -status "Starting $($_.Label)..."
                     $serviceToStart | start-vmhostservice
                     if ($onboot) {write-progress -id 1 -activity "Modifying Remote CLI on $VMHost" -status "Setting $($_.Label) to start up automatically..."
                         $serviceToStart | set-vmhostservice -policy "On"
                     }
                 } else {
                     write-progress -id 1 -activity "Modifying Remote CLI on $VMHost" -status "Stopping $($_.Label)..."
                     $serviceToStart | stop-vmhostservice -confirm:$false
                     if ($onboot) {write-progress -id 1 -activity "Modifying Remote CLI on $VMHost" -status "Setting $($_.Label) to not start up with host..."
                         $serviceToStart | set-vmhostservice -policy "Off"
                     }
                 }
             }
         }
     }
 }
`

