---
Author: hellorobo
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: utf-8
License: cc0
PoshCode ID: 6257
Published Date: 2016-03-18t09
Archived Date: 2016-08-17t11
---

# restart wifi and say it - 

## Description

restart wireless service and say it

## Comments



## Usage



## TODO



## function

`speak-text`

## Code

`#
 #
 function Speak-Text
 {
     [CmdletBinding()]
     [OutputType([int])]
     Param
     (
         [Parameter(Mandatory=$true,
                    ValueFromPipelineByPropertyName=$true,
                    Position=0)]
         [string]$text
     )
 
     Begin
     {
         $speaker = New-Object -com SAPI.SpVoice
     }
     Process
     {
         $speaker.Speak($text)
     }
     End
     {
         $speaker.Dispose()
     }
 }
 
 Write-Output "Checking if Wi-Fi is working ... ?"
 $Test = (Test-Connection -Quiet plkon-nt7000)
 Write-Output "... $Test"
 if ($Test -eq $False) {
     Speak-Text "Och nie, chyba nie mam po³¹czenia do systemu N te 7000"
     Restart-NetAdapter -Name Wi-Fi
     Start-Sleep 5
     netsh wlan connect name=mcn-forklift 
     Speak-Text "Nie martw siê, pracuj dalej, zrestartowa³am za ciebie po³¹czenie z bezprzewodówk¹, bo wiem ¿e masz wa¿niejsze sprawy na g³owie"
 
 }
 
`

