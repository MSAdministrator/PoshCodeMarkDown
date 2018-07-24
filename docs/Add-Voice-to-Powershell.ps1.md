---
Author: mike hays
Publisher: 
Copyright: 
Email: 
Version: 1.1 (11 nov 2008)
Encoding: ascii
License: cc0
PoshCode ID: 667
Published Date: 2009-11-11t12
Archived Date: 2017-03-18t15
---

# add voice to powershell - 

## Description

add voice to powershell

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###
 ###
 
 $spokenText = "Super ca li fragilistic expi alidocious"
 
 $voice = New-Object -com SAPI.SpVoice
 
 $voiceList = $voice.GetVoices()
 
 $voiceDescList = @()
 for ($i=0; $i -lt $voiceList.Count; $i++)
 {
     $voiceDescList += $voiceList.Item($i).GetDescription()
 }
 
 if ($voiceDescList -contains "LH Michelle")
 {
     $voiceMember = "Name=LH Michelle"
 }
 else
 {
     $voiceMember = "Name=" + $voiceDescList[0]
 }
 $voiceToUse = $voice.GetVoices($voiceMember)
 
 $voice.Voice = $voiceToUse.Item(0)
 
 [void] $voice.Speak($spokenText)
 
`

