---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5248
Published Date: 
Archived Date: 2014-06-25t05
---

#  - 

## Description

check all active directory computers for ip address

## Comments



## Usage



## TODO



## function

`get-ipfromad`

## Code

`#
 #
 function Get-IPFromAD {
 	
     import-module ActiveDirectory
     $PATH = "C:\Reports\$(get-date -UFormat "%Y%m%d%A-%H%M")-AD_DNS_Report.csv"
     get-adcomputer -filter * | % { 
         
         $computer = $_.Name
         $record = @{"Computer" = $computer}
 
         try {
             $ip = [System.Net.Dns]::GetHostEntry($computer).AddressList.IPAddressToString
             $record.Add("IPAddress", $ip)        
         }
 
         catch {
             $record.Add("IPAddress", "NULL")
         }
             New-Object PSObject -property $record
        
        } | export-csv -path $PATH
 
 }
`

