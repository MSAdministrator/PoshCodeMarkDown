---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 525
Published Date: 
Archived Date: 2009-01-04t02
---

# automount new psdrives - 

## Description

this only works powershell v2.0 ctp2, and you�ll need to save it as automount.psm1 in a directory under your documents folder like so (vista example)

## Comments



## Usage



## TODO



## class

`enable-automount`

## Code

`#
 #
   
 $query = new-object System.Management.WqlEventQuery   
 $query.EventClassName = "__InstanceOperationEvent"  
   
 $query.WithinInterval = new-object System.TimeSpan 0,0,2   
   
 $query.QueryString = "Select * from Win32_VolumeChangeEvent"  
   
 $watcher = new-object System.Management.ManagementEventWatcher $query  
   
 Register-ObjectEvent $watcher -EventName "EventArrived" `   
     -SupportEvent "WMI.VolumeChange" -Action {   
         & (get-module automount) VolumeChangeCallback @args  
     }   
   
   
 $eventTypes = @{   
     1 = "ConfigurationChanged";   
     2 = "Arrived";   
     3 = "Removed";   
     4 = "Docking";   
 }   
   
 function VolumeChangeCallback ($sender, $eventargs) {   
     trap { write-warning $_ }   
   
     $driveName = $eventArgs.NewEvent.DriveName.TrimEnd(":")   
   
     $forwardedEvent = "Device$($eventTypes[$eventType])"  
        
     [void]( New-PSEvent "PowerShell.$forwardedEvent" -Sender $driveName `   
         -EventArguments $eventargs )   
 }   
   
 function Enable-AutoMount {   
   
     Register-PSEvent -SourceIdentifier "PowerShell.DeviceArrived" `   
         -Action {               
             new-psdrive -name $args[0] -psprovider `   
                 filesystem -root "$args[0]:";   
          }   
   
     Register-PSEvent -SourceIdentifier "PowerShell.DeviceRemoved" `   
         -Action {   
         }   
        
     $watcher.Start()   
 }   
   
 function Disable-AutoMount {   
   
     Unregister-PSEvent -SourceIdentifier "PowerShell.DeviceArrived"  
     Unregister-PSEvent -SourceIdentifier "PowerShell.DeviceRemoved"  
        
     $watcher.Stop()   
 }   
   
 Export-ModuleMember Enable-AutoMount, Disable-AutoMount   
   
 Enable-AutoMount
`

