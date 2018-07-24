---
Author: don jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5255
Published Date: 2014-06-20t17
Archived Date: 2014-06-26t08
---

# corptools module - 

## Description

corptools.psm1

## Comments



## Usage



## TODO



## function

`get-corpnetinfo`

## Code

`#
 #
 function Get-CorpNetInfo {
     [cmdletbinding()]
     param(
         [Parameter(Mandatory,ValueFromPipeline)]
         [string[]]$ComputerName
     )
     PROCESS {
         foreach ($computer in $computername) {
             $session = New-CimSession -ComputerName $computer
 
             $adapters = Get-NetAdapter -CimSession $session -Physical
             foreach ($adapter in $adapters) {
 
                 $addresses = Get-NetIPAddress -CimSession $session -InterfaceIndex ($adapter.ifIndex)
                 foreach ($address in $addresses) {
 
                     $output = @{'ComputerName'=$computer;
                                 'Adapter'     =$adapter.name;
                                 'ifIndex'     =$adapter.ifIndex;
                                 'Speed'       =$adapter.LinkSpeed;
                                 'IPAddress'   =$address.IPAddress;
                                 'Subnet'      =$address.Prefixlength}
                     $obj = New-Object -TypeName PSObject -Property $output
                     $obj.psobject.typenames.insert(0,'Corp.NetInfo')
                     Write-Output $obj
 
 
             $session | Remove-CimSession
 
`

