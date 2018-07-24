---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4055
Published Date: 2013-03-27t23
Archived Date: 2013-03-30t05
---

# speaktome - 

## Description

msagent speech script with voice selection and interluded speaking

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 inspired by HelloKitty script and Mike Hays's obsession for Julie Andrews
 added voice selection menu, should work for everyone anywhere
 interluded speaking by using string array joined with newline
 still need to figure out $voiceToUse code, it's working but I don't know why or how
 https://github.com/chriskenis/POSH.git
 #>
 
 [CmdletBinding()]
 param(
 [string] $Name = [Environment]::UserName,
 [string[]] $Greetings = @("This is your computer speaking","It is now $(Get-Date)","The end"),
 [byte] $Speed = 1
 )
 
 process{
 $message = @"
         .-. __ _ .-.
         |  `  / \  |
         /     '.()--\
        |         '._/
       _| O   _   O |_
       =\    '-'    /=
         '-._____.-'
         /`/\___/\`\
        /\/o     o\/\
       (_|         |_)
         |____,____|
         (____|____)
 		
 	Hello World
 "@
 Write-Host $message -foregroundcolor magenta
 $voice = new-object -com SAPI.SpVoice
 $voice.Voice =  $(SelectVoice $voice)
 $voice.Rate = $Speed
 
 }
 
 begin{
 $nl = [Environment]::NewLine
 
 Function SelectVoice($VoiceCom){
 $voiceList = $VoiceCom.GetVoices()
 $voiceDescList = @()
 For ($i=0; $i -lt $voiceList.Count; $i++){
 	$voiceDescList += $($voiceList.Item($i).GetDescription())
 	write-verbose "$voiceDescList[$i]"
 	}
 Write-Host "$nl Voice Selection: $nl"
 For ($i=0; $i -lt $voiceList.Count; $i++){Write-Host -ForegroundColor Green "$i --> $($voiceDescList[$i])"}
 Write-Host -ForegroundColor Green "q --> quit script"
 Do {
 	$SelectIndex = Read-Host "Select voice by number or 'enter' (=default voice)"
 	Switch -regex ($SelectIndex){
 		"^q.*" 	{$SelectIndex="quit"; $kip = $true}
 		"\d" 	{$SelectIndex = $SelectIndex -as [int];$kip = $false}
 		"^\s*$" {$SelectIndex=0; $kip = $false}
 		}
 	}
 Until (($SelectIndex -lt $voiceList.Count) -OR $SelectIndex -like "q*")
 If ($kip) {exit}
 $voiceMember = "Name=" + $voiceDescList[$SelectIndex]
 write-verbose "$voiceMember"
 $voiceToUse = $VoiceCom.GetVoices($voiceMember.Id).Item($SelectIndex)
 write-verbose "Voice to use = $($voiceToUse.Id)"
 return $voiceToUse
 }
 
 }
 
 end{
 [void]$voice.speak("Hi $Name, " + $nl + [string]::join($nl,$Greetings))
 }
`

