---
Author: patrick
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1630
Published Date: 2010-02-08t03
Archived Date: 2016-05-26t20
---

# check-nsca.ps1 - 

## Description

sends with nsca (nagios client) all status informations over vms

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $statvalues=("mem.usage.average", "cpu.usage.average")
 $nsca_stat = ""
 [int]$warnlevel = 85
 [int]$criticallevel = 90
 $status = ""
 $nagsrv = "nagios-srv.local"
 
 $vms = Get-VM | Where-Object { $_.PowerState -eq "PoweredOn" } | sort-object
 
 foreach ($vm in $vms) {
 	$statvalues | foreach {
 		[int]$statavg = ($vm | Get-Stat -Stat $_ -Start ((get-date).AddMinutes(-5)) -MaxSamples 500 | Measure-Object -Property Value -Average).Average
 		$vmdns = ($vm | Get-VMGuest).Hostname
 		switch ($_) {
 			"mem.usage.average" { $nsca_stat = "mem_vm"; $desc = "Memory Usage" }
 			"cpu.usage.average" { $nsca_stat = "cpu_vm"; $desc = "CPU Usage" }
 		}
 		if ($statavg -gt $criticallevel) {
 			$status = "2"
 			$desc = "CRITICAL: " + $desc
 		} elseif ($statavg -gt $warnlevel) {
 			$status = "1"
 			$desc = "WARNING: " + $desc
 		} elseif ($statavg -lt $warnlevel) {
 			$status = "0"
 		}
 		$nsca = "${vmdns};${nsca_stat};${status};${desc} ${statavg}% | ${nsca_stat}=${statavg};$warnlevel;$criticallevel;0;100"
 		Write-Host $nsca
 		if ($vmdns) { echo $nsca | ./send_nsca.exe -H $nagsrv -c send_nsca.cfg -d ";" }
 	}
 }
`

