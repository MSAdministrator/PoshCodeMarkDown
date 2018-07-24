---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 565
Published Date: 
Archived Date: 2008-09-15t21
---

#  - 

## Description

this function calculates cpu usage through wmi object

## Comments



## Usage



## TODO



## function

`cpu-usage`

## Code

`#
 #
 function cpu-usage
  { if ($Args)
         {$machine=$Args}
     else
         {$machine="localhost"}
       
  while($auxiliary -ne 'Q')
        {
 
 $before   = gwmi win32_perfrawdata_perfproc_process -ComputerName $machine
 sleep -Milliseconds (100) 
 
 $after = gwmi win32_perfrawdata_perfproc_process -ComputerName $machine
 
 $difference = @{}
 $result = @{}
 
 
 foreach ($process_before in $before)
 { foreach ($process_after in $after)
     { if ($process_after.name -eq $process_before.name)
         {$difference_process = [long]$process_after.percentprocessortime -[long]$process_before.percentprocessortime
          $difference[$process_before.name] = $difference_process      
         }
     }
 }
 
 $sum = $difference["_Total"]
 
 foreach ($i in $difference.keys)
     {$result[$i] = ((($difference.$i)/$sum)*100)
     }
 
           
 $result= (($result.GetEnumerator() | sort value -Descending)[1..10])
 
   
       Clear-Host
       Write-Host ""
       Write-Host "press Q to quit, another key to refresh"
     
      format-table -AutoSize -InputObject $result @{Label= "Name:"; Expression = {$_.name}},`
         @{Label = "percentaje CPU"; Expression= {"{0:n2}" -f ($_.Value)}}
       
         $auxiliary = $Host.UI.RawUI.ReadKey()
         $auxiliary = [string]$auxiliary.character
         $auxiliary=$auxiliary.toupper()       
         }      
 }
`

