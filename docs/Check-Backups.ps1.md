---
Author: gonads99
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6852
Published Date: 2017-04-21t04
Archived Date: 2017-04-27t07
---

# check backups - 

## Description

check for successful backups in networker

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $devList = New-Object System.Collections.ArrayList
 $prodList = New-Object System.Collections.ArrayList
 
 Try {
     $hostList = gc $hostsToCheckFile -ErrorAction Stop
 }
 Catch {
     #$ErrorMessage = $_.Exception.Message
     #$FailedItem = $_.Exception.ItemName
     echo "ERROR: Could not open $hostsToCheckFile..."
     Break
 }
     
 
 foreach ($line in $hostList) {
 
     if ($line -match "^nzp") {
         echo "Adding $line to prod list..."
         $prodList.Add($line)
     }
     elseif ($line -match "^nz") {
         echo "Adding $line to dev list..."
         $devList.Add($line)
     }
     else {
         echo "Cannot process $line..."
     }
 }
 
 $s = New-PSSession -ComputerName $devServer #-Credential $devCreds
 
 Invoke-Command -Session $s -ArgumentList (,$devList) -ScriptBlock {
     param($devList)
 
     #$devList
 
     foreach ($client in $devList) {
         echo "Checking client $client..."
         $out = (mminfo -q "savetime>=24 hours ago,name=/,client=$client" -ot -r "savetime,client,name,sumsize,level")
         echo $out
     }
 }
 
 
 echo "Removing PS session..."
 Remove-PSSession $s
`

