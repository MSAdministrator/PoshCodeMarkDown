---
Author: vince ypma
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6001
Published Date: 2016-09-04t01
Archived Date: 2016-05-17t10
---

# using read-choice - 

## Description

i saved joel bennett’s read-choice function as a module and wrote this function.  i wanted to make it useful (especially to beginners) so it doesn’t look like a “normal” menu script, but bennett’s function is so friendly that you could write a “normal” menu script without using variables at all.  to create the module save read-choice in a file named readchoice.psm1 and save it in $psmodulepath\readchoice.

## Comments



## Usage



## TODO



## function

`invoke-powertipsmenu`

## Code

`#
 #
 function Invoke-PowerTipsMenu
 {
     function Start-PowerTipsMonthly ([int]$Volume)
     {
         [string]$urlHead = "http://powershell.com/cs/media/p"
         [string]$urlTail = "download.aspx"
         [int[]]$urlIndex = @(0, 23856, 24814, 25742, 26784, 28283, 29098,
                                 29920, 30542, 31297, 32274, 33169, 38383)
 
         Start-Process -FilePath "$urlHead/$($urlIndex[$Volume])/$urlTail"
     }
 
     Import-Module -Name ReadChoice
 
     [string]$title  = "`nPowerTips Monthly`n" + ("="*17)
     [string]$prompt = "`nSelect PowerTips Monthly Volume(s)`n" + ("-"*34)
 
     "&File System Tasks",
     "&Arrays and Hash Tables",
     "&Date, Time, and Culture",
     "&Objects and Types",
     "&WMI",
     "Regular &Expressions",
     "F&unctions",
     "Static .&NET Methods",
     "&Registry",
     "&Internet-Related Tasks",
     "&XML-Related Tasks",
     "&Security-Related Tasks",
     "&Quit" | 
         Read-Choice -Title $title -Prompt $prompt -MultipleChoice |
             ForEach-Object {
                 if (0..11 -contains $_)
                 {
                     Start-PowerTipsMonthly -Volume ($_ + 1)
                 }
             }
 
     Remove-Module -Name ReadChoice
 }
`

