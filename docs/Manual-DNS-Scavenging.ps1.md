---
Author: stephen wheet
Publisher: 
Copyright: 
Email: 
Version: 1.2
Encoding: ascii
License: cc0
PoshCode ID: 2079
Published Date: 2011-08-17t16
Archived Date: 2017-02-16t15
---

# manual dns scavenging - 

## Description

comment

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #==========================================================================
 #
 #
 #
 #
 #
 #==========================================================================
     
 $NetworkRange = "192.168.1.*","192.168.2.*"
 
 $TotalAgingInterval = 60
 
 
 ForEach ($Network in $NetworkRange){
     $filename = "DC-" + $ServerName + "--DOMAIN-" + $ContainerName + "--AGE-" + $TotalAgingInterval + `
 "--RANGE-" + $Network.Replace("*","") + ".csv"
     $logfile = "D:\reports\DNSscavenge\$filename"
     $list = "Ownername,TimeStamp,Deleted"
     $list | format-table | Out-File "$logfile"
 
 $MinTimeStamp = [Int](New-TimeSpan `
   -Start $(Get-Date("01/01/1601 00:00")) `
   -End $((Get-Date).AddDays(-$TotalAgingInterval))).TotalHours
 
 $records = Get-WMIObject -Computer $ServerName `
   -Namespace "root\MicrosoftDNS" -Class "MicrosoftDNS_AType" `
   -Filter `
   "ContainerName='$ContainerName' AND TimeStamp<$MinTimeStamp AND TimeStamp<>0 " 
 
 ForEach ($record in $records){
     $IPA = $record.IPAddress
     
     ForEach ($Network in $NetworkRange){
         
         If ( $IPA -like $Network ){
         
             $Ownername = $record.Ownername
             $TimeStamp = (Get-Date("01/01/1601")).AddHours($record.TimeStamp)
             Write-host "$Ownername,$IPA,$TimeStamp"
             $filename = "DC-" + $ServerName + "--DOMAIN-" + $ContainerName + "--AGE-" + $TotalAgingInterval + `
 "--RANGE-" + $Network.Replace("*","") + ".csv"
          
             If ($DeleteKey){
                 $record.psbase.Delete()
                 
                 If($?) { 
                     
                     Write-host "Successfully deleted A record: $Ownername"
                                
                 }Else { 
                     Write-host "Could not delete A record: $Ownername, error: $($error[0])"
                 }
             
             $list = ("$Ownername,$TimeStamp,$?")
             $list | format-table | Out-File -append "$logfile"
         
             }Else{
         
             $list = ("$Ownername,$TimeStamp,No")
             $list | format-table | Out-File -append "$logfile" 
                
`

