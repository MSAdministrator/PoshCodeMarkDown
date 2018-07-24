---
Author: mitch
Publisher: 
Copyright: 
Email: 
Version: 10.200
Encoding: ascii
License: cc0
PoshCode ID: 3149
Published Date: 2012-01-06t15
Archived Date: 2016-03-03t19
---

# update subnet masks - 

## Description

script to update subnet mask for all devices connected to a specific network.  specify network and new subnet mask and run on each device.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
 #!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
 $network = "10.200.*"
 $newSubnet = "255.255.240.0"
 
 $nics = Get-WMIObject Win32_NetworkAdapterConfiguration | where { $_.IPEnabled -eq "TRUE" }
 
 foreach ($nc in $nics) 
 { 
     $addresses = [System.Collections.ArrayList]$nc.IPAddress
     $smasks = [System.Collections.ArrayList]$nc.IPSubnet
     $needsChange = $FALSE
     
     write-host ("{0} addresses found" -f $addresses.count)
     
     for ($i = 0; $i -lt $addresses.count;)
     {
         $addy = $addresses[$i]
         $sm = $smasks[$i]
 
         if ($addy -notLike '*.*.*.*')
         {
             $addresses.RemoveRange($i, 1)
             $smasks.RemoveRange($i, 1)
         }    
         elseif ($addy -like $network -and $sm -ne $newSubnet)
         {
             write-host ("Updating address {0}/{1}" -f $addy, $sm)
             $smasks[$i] = $newSubnet
             $needsChange = $TRUE
             
             $i++;
         }
         else
         {
             write-host ("Skipping address {0}/{1}" -f $addy, $sm)
             
             $i++;
         }
     }
     
     if ($needsChange)
     {
         write-host ("Updating Addresses...")
         $res = [int]($nics.EnableStatic($addresses, $smasks).ReturnValue)
         
         if ($res -eq 0) {
             write-host "Updated Successfully!"
         } else {
             write-host ("Updating the IP Address failed with error {0}" -f $res)
         }
     }
     else 
     {
         write-host "No changes needed"
     }
 }
 write-host 'done '
`

