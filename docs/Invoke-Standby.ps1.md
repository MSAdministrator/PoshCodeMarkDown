---
Author: tjdot
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2280
Published Date: 2011-10-04t12
Archived Date: 2015-07-20t23
---

# invoke-standby - 

## Description

collection of functions to shutdown, reboot, logoff, standby(sleep) or hibernate your computer.

## Comments



## Usage



## TODO



## function

`invoke-shutdown`

## Code

`#
 
 function Invoke-Shutdown
 {
     &"$env:SystemRoot\System32\shutdown.exe" -s
 }
 function Invoke-Reboot
 {
     &"$env:SystemRoot\System32\shutdown.exe" -r
 }
 function Invoke-Logoff
 {
     &"$env:SystemRoot\System32\shutdown.exe" -l
 }
 Set-Alias lg Invoke-Logoff
 
 function Invoke-Standby
 {
     &"$env:SystemRoot\System32\rundll32.exe" powrprof.dll,SetSuspendState Standby
 }
 Set-Alias csleep Invoke-Standby
 
 function Invoke-Hibernate
 {
     &"$env:SystemRoot\System32\rundll32.exe" powrprof.dll,SetSuspendState Hibernate
 }
`

