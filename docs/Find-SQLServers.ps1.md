---
Author: jgrote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5785
Published Date: 2016-03-13t18
Archived Date: 2016-03-04t23
---

# find-sqlservers - 

## Description

this code searches your ad for servers that have active computer accounts within the last 30 days, pings them, and returns a list of service objects for installed sql instances.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 
 $servers = get-adcomputer -filter {operatingsystem -like "Windows Server*"} -properties lastlogondate | where {$_.lastlogondate -gt (get-date).adddays(-30)}
 
 $sqlservices = foreach ($server in $servers) {
         if (test-connection $server.dnshostname -quiet -count 1 -delay 1) {
             Get-Service -verbose -computername $server.dnshostname -name "MSSQL*"
 
 $sqlinstances = $sqlservices | where {$_.servicename -like "MSSQLServer" -or $_.servicename -like "MSSQL$*"}
 
 $sqlinstances
`

