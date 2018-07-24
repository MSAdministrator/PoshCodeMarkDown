---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4028
Published Date: 2013-03-18t17
Archived Date: 2013-03-21t06
---

# get-process -eq pslist - 

## Description

you can sort get-process cmdlet output in pslist style, of course if you has administrator privileges

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ps | % -b {$arr = @()} -p {
   $str = "" | select Name, PID, Time
   $str.Name = $_.ProcessName
   $str.PID  = $_.Id
   $str.Time = $(try {$_.StartTime} catch {return [DateTime]::MinValue})
   $arr += $str
 } -end {$arr | sort Time | ft -a}
`

