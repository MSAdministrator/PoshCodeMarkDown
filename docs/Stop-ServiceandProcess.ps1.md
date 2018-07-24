---
Author: daniel cheng
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5607
Published Date: 2015-11-20t18
Archived Date: 2015-01-31t20
---

# stop-serviceandprocess - 

## Description

stop service and associated pid. sometimes homegrown win32 services has its pid still terminating gracefully in the background for quite a while. this function allows to wait indefinitely, or after some time, kill the associated pid.

## Comments



## Usage



## TODO



## function

`stop service and its associated processid (pid).`

## Code

`#
 #
 
 function Stop-ServiceandProcess {
     param (
         [string[]]$servers = 'localhost',
         [parameter(Mandatory)][string[]]$services,
         [int]$killDelay = $null,
         [string]$terseIndicator = '.',
         [switch]$showEndStatus = $false
     )
 
 
     $serviceState = @{
         00 = 'The request was accepted.';
         01 = 'The request is not supported.';
         02 = 'The user did not have the necessary access.';
         03 = 'The service cannot be stopped because other services that are running are dependent on it.';
         04 = 'The requested control code is not valid, or it is unacceptable to the service.';
         05 = 'The requested control code cannot be sent to the service because the state of the service (Win32_BaseService State property) is equal to 0, 1, or 2.';
         06 = 'The service has not been started.';
         07 = 'The service did not respond to the start request in a timely fashion.';
         08 = 'Unknown failure when starting the service.';
         09 = 'The directory path to the service executable file was not found.';
         10 = 'The service is already running.';
         11 = 'The database to add a new service is locked.';
         12 = 'A dependency this service relies on has been removed from the system.';
         13 = 'The service failed to find the service needed from a dependent service.';
         14 = 'The service has been disabled from the system.';
         15 = 'The service does not have the correct authentication to run on the system.';
         16 = 'This service is being removed from the system.';
         17 = 'The service has no execution thread.';
         18 = 'The service has circular dependencies when it starts.';
         19 = 'A service is running under the same name.';
         20 = 'The service name has invalid characters.';
         21 = 'Invalid parameters have been passed to the service.';
         22 = 'The account under which this service runs is either invalid or lacks the permissions to run the service.';
         23 = 'The service exists in the database of services available from the system.';
         24 = 'The service is currently paused in the system.';
     }
 
     $terminateReturnCode = @{
         00 = 'Successful Completion';
         02 = 'Access Denied';
         03 = 'Insufficient Privilege';
         08 = 'Unknown Failure';
         09 = 'Path Not Found';
         21 = 'Invalid Parameter';
     }
 
     $acceptableConditions = [ordered]@{
         AcceptStop = 'True';
         Status = 'OK';
         State = @('Running', 'Paused');
         StartMode = @('Auto','Manual');
     }
 
     filter Get-FormattedTimestamp {return Get-Date -Format s}
 
     foreach ($server in $servers){
         foreach ($service in $services){
             $currentService = gwmi -ComputerName $server -Class win32_service -Filter "name='$service'" -ea SilentlyContinue
             $processId = $currentService.ProcessId
 
             if ($currentService -isnot [object]){
                 Write-Warning "Error with creating server object ($server) for service ($service)!"
             } else {
                 foreach ($condition in $acceptableConditions.Keys){
                     if (-not ($acceptableConditions.$condition -contains $currentService.$condition)){
                         "{0}: {1} -- {2}: service ({3}, pid={4}); message: {5}" -f (Get-FormattedTimestamp), $localhost, $server, $service, $processId, "Error! $($currentService.Name).$condition = '$($currentService.$condition)'.. skipping!"
                         $condition = $false
                         break
                     }
                 }
             }
             if ($condition){
                 "{0}: {1} -> {2}: service ({3}, pid={4}); message: {5}" -f (Get-FormattedTimestamp), $localhost, $server, $service, $processId, 'Stop command sent.'
                 "{0}: {1} <- {2}: service ({3}, pid={4}); message: {5}" -f (Get-FormattedTimestamp), $localhost, $server, $service, $processId, $serviceState.$([int]$currentService.StopService().ReturnValue)
 
                 $timeElapsed = [System.Diagnostics.Stopwatch]::StartNew()
                 do {
                     $exitCode = $([int]$currentService.InterrogateService().ReturnValue)
                     if ($terse){
                         Write-Host -NoNewline $terseIndicator
                     } else {
                         "{0}: {1} -> {2}: service ({3}, pid={4}); message: {5}" -f (Get-FormattedTimestamp), $localhost, $server, $service, $processId, "InterrogateService()"
                         "{0}: {1} <- {2}: service ({3}, pid={4}); message: {5}" -f (Get-FormattedTimestamp), $localhost, $server, $service, $processId, "Exit code = $exitCode, $($serviceState.$exitCode)"
                     }
                     if (([bool]$killDelay) -and ($timeElapsed.Elapsed.Seconds -ge $killDelay)){
                         "{0}: {1} -> {2}: service ({3}, pid={4}); message: {5}" -f (Get-FormattedTimestamp), $localhost, $server, $service, $processId, "Time threshold ($killDelay second(s)) expired, sending kill command."
                         $terminateStatus = (gwmi -ea SilentlyContinue -ComputerName $server -Class win32_process -Filter "handle='$processId'").Terminate()
                     }
                     Start-Sleep -Seconds $InterrogateServiceDelay
                 } until (
                     ([bool]!(Get-Process -ComputerName $server -Id $currentService.ProcessId -ea SilentlyContinue).Id) -and
                     ((gwmi -ComputerName $server -Class win32_service -Filter "name='$service'" -ea SilentlyContinue).State -eq 'Stopped')
                 )
                 "{0}: {1} <- {2}: service ({3}, pid={4}); message: {5}" -f (Get-FormattedTimestamp), $localhost, $server, $service, $processId, $terminateReturnCode.([int]$($terminateStatus.ReturnValue))
                 "{0}: {1} -- {2}: service ({3}, pid={4}); message: {5}" -f (Get-FormattedTimestamp), $localhost, $server, $service, $processId, "Operation took $($timeElapsed.Elapsed.TotalSeconds) seconds."
                 $timeElapsed.Stop()
             }
             if ($showEndStatus){gwmi -ComputerName $server -Class win32_service -Filter "name='$service'" | ft PSComputerName, Name, ServiceType, StartMode, State, ProcessId, PathName -AutoSize}
         }
     }
 }
`

