---
Author: jgrote
Publisher: 
Copyright: 
Email: 
Version: 4.0
Encoding: ascii
License: cc0
PoshCode ID: 1955
Published Date: 2011-07-08t10
Archived Date: 2015-01-28t08
---

# esxi config backup scrip - 

## Description

**********edit

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 **********EDIT: See Carter's Example, it's way simpler: http://poshcode.org/1559**********
 
 
 
 
 $vSphereCLIPath = "C:\Program Files (x86)\VMware\VMware vSphere CLI\"
 $BackupDest = "E:\Backup\ESXi"
 #
 $ESXiCSV = "C:\Scripts\ESXiBackupList.csv"
 
 $eventsource = "Backup-ESXi"
 $eventlog = "Application"
 if (!(get-eventlog -logname $eventlog -source $eventsource)) {new-eventlog -logname $eventlog -source $eventsource}
 trap {write-eventlog -logname $eventlog -eventID 1 -source $eventsource -EntryType "Error" -Message "An Error occured during $($eventsource): $_ . Script run from $($MyInvocation.MyCommand.Definition)"; exit}
 
 $ESXiCSVIsPresentTest = Get-Item $ESXiCSV
 
 
 
 import-csv $ESXiCSV | ForEach-Object {
     $BackupResult = invoke-expression "& '$vSphereCLIPath\perl\bin\perl.exe' '$vSphereCLIPath\bin\vicfg-cfgbackup.pl' --server $($_.HOSTNAME) --username $($_.USERNAME) --password $($_.PASSWORD) --save $($BackupDest)\$($_.HOSTNAME)-cfgbackup.tgz"
     if ($LastExitCode -ne 0) {throw "An Error occurred while executing $BackupBin for $($_.HOSTNAME): $BackupResult"}
 }
 
 $successText = "ESXi Configurations were successfully backed up to $BackupDest. Script run from $($MyInvocation.MyCommand.Definition)""
 write-eventlog -logname $eventlog -eventID 1 -source $eventsource -EntryType "Information" -Message $successText
`

