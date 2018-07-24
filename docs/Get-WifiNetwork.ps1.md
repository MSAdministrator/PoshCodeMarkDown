---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6607
Published Date: 2016-11-04t03
Archived Date: 2016-11-08t01
---

# get-wifinetwork - 

## Description

get-wifinetwork – return the parsed output of netsh wlan show network mode=bssid in psobject form. does exactly what it says on the tin. requires vista/2008 or higher, or xp sp3 with a hotfix (i can’t recall which one, sorry.)

## Comments



## Usage



## TODO



## function

`get-wifinetwork`

## Code

`#
 #
 function Get-WifiNetwork {
  end {
   netsh wlan sh net mode=bssid | % -process {
     if ($_ -match '^SSID (\d+) : (.*)$') {
         $current = @{}
         $networks += $current
         $current.Index = $matches[1].trim()
         $current.SSID = $matches[2].trim()
     } else {
         if ($_ -match '^\s+(.*)\s+:\s+(.*)\s*$') {
             $current[$matches[1].trim()] = $matches[2].trim()
         }
     }
   } -begin { $networks = @() } -end { $networks|% { new-object psobject -property $_ } }
  }
 }
 
 ps> Get-WifiNetwork| select index, ssid, signal, 'radio type' | sort signal -desc | ft -auto
`

