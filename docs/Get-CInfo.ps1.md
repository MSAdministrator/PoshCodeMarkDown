---
Author: tonybob
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3946
Published Date: 2013-02-14t04
Archived Date: 2013-02-19t01
---

# get-cinfo - 

## Description

this function allows you to search ad for computers by computer name. once you find the computer you want it displays info about that computer to you. the info is

## Comments



## Usage



## TODO



## function

`get-cinfo`

## Code

`#
 #
 function Get-CInfo {
     param($Comp)
 
   Function PC-Name{
 param ($compname)
 $ADS = Get-ADComputer -Filter {name -eq $compname} -Properties * | Select-Object -Property name
 If($ads -eq $null) {$false}
 ELSE{$True}
                 }
 
 $ping = PC-Name $COMP
 
 if ($ping -eq $true) {
 Write-Host -ForegroundColor Green ("Computer Info For: $COMP")
 
 
 $a= @{Expression={$_.Name};Label="Comp. Name";width=25},
     @{Expression={$_.ipv4address};Label="IP Address";width=25},
     @{Expression={$_.operatingsystem};Label="Operating System";width=25},
     @{Expression={$_.Created};Label="Created";width=25},
     @{Expression={$_.lastlogondate};Label="Last Logon";width=25},
 
 Get-ADComputer -f {name -eq $comp} -Properties * |
 select-object -property name,ipv4address,operatingsystem,created,lastlogondate,logoncount |
 format-table -Wrap -AutoSize $A
                         }
 
 ELSEIF ($ping -eq $false) {
                             do { Write-Host -ForegroundColor yellow "No Match Found, Below Is An AD Search, Try Again (CTRL-C to Exit):"
                                 $SEARCH = "*$COMP*"
                                 Get-ADComputer -Filter {name -like $SEARCH} | Select-Object -property name,enabled | Format-Table -AutoSize
                                 $comp = (read-host "What is the Computer Name? (EX. demersm-l7567)").ToLower()
                                 function Ping-Server {
                                                      Param([string]$srv)
                                                      $pingresult = Get-WmiObject Win32_PingStatus -Filter  "Address='$srv'"
                                                      if($pingresult.StatusCode -eq 0) {$true} else {$false}
                                                      }
                               
                                 $PING2 = PC-Name $COMP
                               }
                             While ($PING2 -eq $false)
                             }
 If($ping2 -eq $true){
 Write-Host -ForegroundColor Green ("Computer Info For: $COMP")
 
 
 $a= @{Expression={$_.Name};Label="Comp. Name";width=25},
     @{Expression={$_.ipv4address};Label="IP Address";width=25},
     @{Expression={$_.operatingsystem};Label="Operating System";width=25},
     @{Expression={$_.Created};Label="Created";width=25},
     @{Expression={$_.lastlogondate};Label="Last Logon";width=25},
 
 Get-ADComputer -f {name -eq $comp} -Properties * |
 select-object -property name,ipv4address,operatingsystem,created,lastlogondate,logoncount |
 format-table -Wrap -AutoSize $A
                         }
                         }
`

