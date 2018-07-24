---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 46.4
Encoding: ascii
License: cc0
PoshCode ID: 1119
Published Date: 
Archived Date: 2009-05-27t10
---

# apache log browser count - 

## Description

a powershell implementation of commandline fu episode 38, i skipped the first step of looking only at posts to login just because i don’t have anything like that in my logs

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Select-String "(MSIE [0-9]\.[0-9]|Firefox\/[0-9]+|Safari|-)" $log -all |
 Group  { $_.matches[-1].Value } | 
 ForEach -begin   { $total = 0 } `
 	-process { $total += $_.Count; $_ } | 
 Sort Count | Select Count, Name |
 Add-Member ScriptProperty Percent { "{0,7:0.000}%" -f (100*$this.Count/$Total) } -Passthru
 
 ##
 
 
 Count Name                                    Percent
 ----- ----                                    -------
    34 Firefox/1                                 2.4% 
   139 MSIE 8.0                                  0.3% 
   290 -                                         1.2% 
   363 Firefox/2                                 3.0% 
   632 MSIE 6.0                                  5.3% 
  1708 Safari                                   14.3%
  3207 MSIE 7.0                                 27.0%
  5534 Firefox/3                                46.4%
`

