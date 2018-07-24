---
Author: glnsize
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 976
Published Date: 2010-03-27t13
Archived Date: 2013-01-24t06
---

# ping-subnet - 

## Description

given an ip address and subnet mask return all alive hosts within that subnet.  uses psjobs as a poor mans threading thus requires v2.

## Comments



## Usage



## TODO



## function

`ping-subnet`

## Code

`#
 #
 Function Ping-Subnet {
     #.Synopsis
     #.Example
     #.Example
     Param(
         [string]
         $IP,
         [string]
         $netmask
     )
     Begin {
         $IPs = New-Object System.Collections.ArrayList
         $Jobs = New-Object System.Collections.ArrayList
         $max = 50
         
         Function Get-NetworkAddress {
             #.Synopsis 
             #.Example 
             Param (
                 [string]
                 $IP, 
                 
                 [string]
                 $Mask, 
                 
                 [switch]
                 $Binary
             )
             Begin {
                 $NetAdd = $null
             }
             Process {
                 $BinaryIP = ConvertTo-BinaryIP $IP
                 $BinaryMask = ConvertTo-BinaryIP $Mask
                 0..34 | %{
                     $IPBit = $BinaryIP.Substring($_,1)
                     $MaskBit = $BinaryMask.Substring($_,1)
                     IF ($IPBit -eq '1' -and $MaskBit -eq '1') {
                         $NetAdd = $NetAdd + "1"
                     } elseif ($IPBit -eq ".") {
                         $NetAdd = $NetAdd +'.'
                     } else {
                         $NetAdd = $NetAdd + "0"
                     }
                 }
                 if ($Binary) {
                     return $NetAdd
                 } else {
                     return ConvertFrom-BinaryIP $NetAdd
                 }
             }
         }
         
         Function ConvertTo-BinaryIP {
             #.Synopsis
             #.Example 
             Param (
                 [string]
                 $IP
             )
             Process {
                 $out = @()
                 Foreach ($octet in $IP.split('.')) {
                     $strout = $null
                     0..7|% {
                         IF (($octet - [math]::pow(2,(7-$_)))-ge 0) { 
                             $octet = $octet - [math]::pow(2,(7-$_))
                             [string]$strout = $strout + "1"
                         } else {
                             [string]$strout = $strout + "0"
                         }   
                     }
                     $out += $strout
                 }
                 return [string]::join('.',$out)
             }
         }
 
 
         Function ConvertFrom-BinaryIP {
             #.Synopsis 
             #.Example 
             Param (
                 [string]
                 $IP
             )
             Process {
                 $out = @()
                 Foreach ($octet in $IP.split('.')) {
                     $strout = 0
                     0..7|% {
                         $bit = $octet.Substring(($_),1)
                         IF ($bit -eq 1) { 
                             $strout = $strout + [math]::pow(2,(7-$_))
                         } 
                     }
                     $out += $strout
                 }
                 return [string]::join('.',$out)
             }
         }
 
         Function ConvertTo-MaskLength {
             #.Synopsis 
             #.Example 
             Param (
                 [string]
                 $mask
             )
             Process {
                 $out = 0
                 Foreach ($octet in $Mask.split('.')) {
                     $strout = 0
                     0..7|% {
                         IF (($octet - [math]::pow(2,(7-$_)))-ge 0) { 
                             $octet = $octet - [math]::pow(2,(7-$_))
                             $out++
                         }
                     }
                 }
                 return $out
             }
         }
 
         Function ConvertFrom-MaskLength {
             #.Synopsis 
             #.Example 
             #.Example 
             Param (
                 [int]
                 $mask
             )
             Process {
                 $out = @()
                 [int]$wholeOctet = ($mask - ($mask % 8))/8
                 if ($wholeOctet -gt 0) {
                     1..$($wholeOctet) |%{
                         $out += "255"
                     }
                 }
                 $subnet = ($mask - ($wholeOctet * 8))
                 if ($subnet -gt 0) {
                     $octet = 0
                     0..($subnet - 1) | %{
                          $octet = $octet + [math]::pow(2,(7-$_))
                     }
                     $out += $octet
                 }
                 for ($i=$out.count;$i -lt 4; $I++) {
                     $out += 0
                 }
                 return [string]::join('.',$out)
             }
         }
         Function Get-IPRange {
             #.Synopsis 
             #.Example 
             #.Example 
             Param (
                 [string]
                 $IP,
                 
                 [string]
                 $netmask
             )
             Process {
                 iF ($netMask.length -le 3) {
                     $masklength = $netmask.replace('/','')
                     $Subnet = ConvertFrom-MaskLength $masklength
                 } else {
                     $Subnet = $netmask
                     $masklength = ConvertTo-MaskLength -Mask $netmask
                 }
                 $network = Get-NetworkAddress -IP $IP -Mask $Subnet 
                 
                 [int]$FirstOctet,[int]$SecondOctet,[int]$ThirdOctet,[int]$FourthOctet = $network.split('.')
                 $TotalIPs = ([math]::pow(2,(32-$masklength)) -2)
                 $blocks = ($TotalIPs - ($TotalIPs % 256))/256
                 if ($Blocks -gt 0) {
                     1..$blocks | %{
                         0..255 |%{
                             if ($FourthOctet -eq 255) {
                                 If ($ThirdOctet -eq 255) {
                                     If ($SecondOctet -eq 255) {
                                         $FirstOctet++
                                         $secondOctet = 0
                                     } else {
                                         $SecondOctet++
                                         $ThirdOctet = 0
                                     }
                                 } else {
                                     $FourthOctet = 0
                                     $ThirdOctet++
                                 }  
                             } else {
                                 $FourthOctet++
                             }
                             Write-Output ("{0}.{1}.{2}.{3}" -f `
                             $FirstOctet,$SecondOctet,$ThirdOctet,$FourthOctet)
                         }
                     }
                 }
                 $sBlock = $TotalIPs - ($blocks * 256)
                 if ($sBlock -gt 0) {
                     1..$SBlock | %{
                         if ($FourthOctet -eq 255) {
                             If ($ThirdOctet -eq 255) {
                                 If ($SecondOctet -eq 255) {
                                     $FirstOctet++
                                     $secondOctet = 0
                                 } else {
                                     $SecondOctet++
                                     $ThirdOctet = 0
                                 }
                             } else {
                                 $FourthOctet = 0
                                 $ThirdOctet++
                             }  
                         } else {
                             $FourthOctet++
                         }
                         Write-Output ("{0}.{1}.{2}.{3}" -f `
                         $FirstOctet,$SecondOctet,$ThirdOctet,$FourthOctet)
                     }
                 }
             }
         }
     }
     Process {
         Get-IPRange $IP $netmask | %{
             [void]$IPs.Add($_)
         }
         While ($IPs.count -gt 0 -or $jobs.count -gt 0) {
             IF ($jobs.count -le $max) {
                 $addjobs = ($max - $jobs.count)
                 Foreach ($IP in ($IPS | Select -first $addjobs)) {
                     [VOID]$Jobs.Add((gwmi -q "SELECT Address,StatusCode FROM Win32_Pingstatus WHERE Address = `'$IP`'" -asjob).Id)
                     $IPs.Remove($IP)
                 }
             }
             $Clean = @()
             
             Foreach ($J in $jobs) {
                 if ((Get-Job -id $j).JobStateInfo.state -eq 'Completed') {
                     write-output (Receive-Job -id $j) | ?{$_.StatusCode -eq 0}| select -expand Address
                     remove-job -id $j
                     $clean += $j
                 }
             }
             Foreach ($c in $Clean) {
                 $jobs.remove($c)
             }
         }
     }
 }
`

