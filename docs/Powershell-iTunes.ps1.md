---
Author: tmaurer
Publisher: 
Copyright: 
Email: 
Version: 1.0.1
Encoding: ascii
License: cc0
PoshCode ID: 2385
Published Date: 
Archived Date: 2010-12-01t23
---

# powershell itunes - 

## Description

powershell itunes is a powershell script which lets you control itunes over powershell.

## Comments



## Usage



## TODO



## function

`start-itunes`

## Code

`#
 #
 #
 #*****************************************************************
 #
 #
 #
 #
 #
 #*****************************************************************
 #
 
 Write-Host "                                                                  " -BackgroundColor Black -ForegroundColor White
 Write-Host "    ____                               __         ____            " -BackgroundColor Black -ForegroundColor White
 Write-Host "   / __ \____ _      _____  __________/ /_  ___  / / /            " -BackgroundColor Black -ForegroundColor White
 Write-Host "  / /_/ / __ \ | /| / / _ \/ ___/ ___/ __ \/ _ \/ / /             " -BackgroundColor Black -ForegroundColor White
 Write-Host " / ____/ /_/ / |/ |/ /  __/ /  (__  ) / / /  __/ / /              " -BackgroundColor Black -ForegroundColor White
 Write-Host "/_/    \____/|__/|__/\___/_/  /____/_/ /_/\___/_/_/               " -BackgroundColor Black -ForegroundColor White
 Write-Host "                                                                  " -BackgroundColor Black -ForegroundColor White                    
 Write-Host "    _ ______                                                      " -BackgroundColor Black -ForegroundColor White
 Write-Host "   (_)_  __/_  ______  ___  _____                                 " -BackgroundColor Black -ForegroundColor White
 Write-Host "  / / / / / / / / __ \/ _ \/ ___/                                 " -BackgroundColor Black -ForegroundColor White
 Write-Host " / / / / / /_/ / / / /  __(__  )                                  " -BackgroundColor Black -ForegroundColor White
 Write-Host "/_/ /_/  \__,_/_/ /_/\___/____/                                   " -BackgroundColor Black -ForegroundColor White                                 
 Write-Host "                                                                  " -BackgroundColor Black -ForegroundColor White
 Write-Host "******************************************************************" -BackgroundColor Black -ForegroundColor White
 Write-Host "                                                                  " -BackgroundColor Black -ForegroundColor White
 Write-Host " Welcome to the Powershell iTunes                                 " -BackgroundColor Black -ForegroundColor White
 Write-Host "                                                                  " -BackgroundColor Black -ForegroundColor White
 Write-Host " (c) 2010 Thomas Maurer www.thomasmaurer.ch/projects              " -BackgroundColor Black -ForegroundColor White
 Write-Host "                                                                  " -BackgroundColor Black -ForegroundColor White
 Write-Host "******************************************************************" -BackgroundColor Black -ForegroundColor White
 Write-Host "                                                                  " -BackgroundColor Black -ForegroundColor White
 Write-Host " for Help use the Get-PSiTunesHelp                                " -BackgroundColor Black -ForegroundColor White
 Write-Host "                                                                  " -BackgroundColor Black -ForegroundColor White
 
 $iTunes = New-Object -ComObject iTunes.Application
 
 
 function Start-iTunes { 
     process {
 	
 		$iTunesRunning = Get-Process | Where-Object {$_.ProcessName -eq "iTunes"}
 		if(!$iTunesRunning) {
 			$script:iTunes = New-Object -ComObject iTunes.Application
 			Write-Host "(PS iTunes): iTunes started" -BackgroundColor Black -ForegroundColor green
 		} 
 		else {
 			Write-Host "(PS iTunes): iTunes allready running" -BackgroundColor Black -ForegroundColor red
 		}
 	}
 }
 
 function Close-iTunes { 
     process {
 	
 		$iTunesRunning = Get-Process | Where-Object {$_.ProcessName -eq "iTunes"}
 		if(!$iTunesRunning) {
 			Write-Host "(PS iTunes): iTunes not running" -BackgroundColor Black -ForegroundColor red
 		} 
 		else {
 			Stop-Process $iTunesRunning.Id
 			Write-Host "(PS iTunes): iTunes closed" -BackgroundColor Black -ForegroundColor red
 		}
 	}
 }
 
 function Play-Track { 
     process {
 		if ($iTunes.Playerstate -eq "0"){
 			$iTunes.Play()
 			Write-Host "(PS iTunes): Now playing:" $iTunes.CurrentTrack.Name "from" $iTunes.CurrentTrack.Artist -BackgroundColor Black -ForegroundColor green
 		} 
 		else {
 			Write-Host "(PS iTunes): Already playing Track:" $iTunes.CurrentTrack.Name "from" $iTunes.CurrentTrack.Artist -BackgroundColor Black -ForegroundColor red
 		}
 	}
 }
 
 function Pause-Track { 
     process {
         if ($iTunes.Playerstate -eq "1"){
 			$iTunes.Pause()	
 			Write-Host "(PS iTunes): iTunes paused" -BackgroundColor Black -ForegroundColor Red
 		}
 		else {
 			Write-Host "(PS iTunes): Not playing any track" -BackgroundColor Black -ForegroundColor red
 		}
 	}
 }
 
 function Resume-Track { 
     process {
     	if ($iTunes.Playerstate -eq "0"){ 
 			$iTunes.Play()
 			Write-Host "(PS iTunes): Now playing:" $iTunes.CurrentTrack.Name "from" $iTunes.CurrentTrack.Artist -BackgroundColor Black -ForegroundColor green
 		}
 		else {
 			Write-Host "(PS iTunes): Already playing Track:" $iTunes.CurrentTrack.Name "from" $iTunes.CurrentTrack.Artist -BackgroundColor Black -ForegroundColor red
 		}
 	}
 }
 
 function Back-Track { 
     process { 
 	$iTunes.BackTrack()
 	Write-Host "(PS iTunes): Now playing:" $iTunes.CurrentTrack.Name "from" $iTunes.CurrentTrack.Artist -BackgroundColor Black -ForegroundColor green	
 	}
 }
 
 function Next-Track { 
     process {   
 	$iTunes.NextTrack()
 	Write-Host "(PS iTunes): Now playing:" $iTunes.CurrentTrack.Name "from" $iTunes.CurrentTrack.Artist -BackgroundColor Black -ForegroundColor green	
 	}
 }
 
 function Get-CurrentTrack { 
     process {    
 	Write-Host "(PS iTunes): Currently playing:" $iTunes.CurrentTrack.Name "from" $iTunes.CurrentTrack.Artist -BackgroundColor Black -ForegroundColor White
 	}
 }
 
 function Mute-iTunes { 
     process {
         if($itunes.Mute -eq $false) {
 			$itunes.Mute = $true
 			Write-Host "(PS iTunes): iTunes muted" -BackgroundColor Black -ForegroundColor Red
 		}
 		else {
 			Write-Host "(PS iTunes): iTunes already muted" -BackgroundColor Black -ForegroundColor red
 		}
 	}
 }
 
 function Unmute-iTunes { 
     process {
     	if($itunes.Mute -eq $true) {
 			$itunes.Mute = $false
 			Write-Host "(PS iTunes): iTunes unmuted" -BackgroundColor Black -ForegroundColor green
 		}
 		else {
 			Write-Host "(PS iTunes): iTunes already unmuted" -BackgroundColor Black -ForegroundColor red
 		}
 	}
 }
 
 function Set-VolumeUP { 
     process {
 		if($iTunes.SoundVolume -lt 100) {
 			$iTunes.SoundVolume = $iTunes.SoundVolume + 2
 			Write-Host "(PS iTunes): iTunes Volume" $iTunes.SoundVolume -BackgroundColor Black -ForegroundColor green	
 		}	
 		else {
 			Write-Host "(PS iTunes): iTunes Volume is already 100%" -BackgroundColor Black -ForegroundColor red
 		}	
 	}
 }
 
 function Set-VolumeDown { 
     process {
 		if($iTunes.SoundVolume -gt 0) {
 			$iTunes.SoundVolume = $iTunes.SoundVolume - 2
 			Write-Host "(PS iTunes): iTunes Volume" $iTunes.SoundVolume -BackgroundColor Black -ForegroundColor red	
 		}	
 		else {
 			Write-Host "(PS iTunes): iTunes Volume is already 0%" -BackgroundColor Black -ForegroundColor red
 		}
 	}
 }
 
 function Set-Volume { 
 	param (
 		[int]$Volume 
 	)
     process {
 		if($Volume -ge 0 -and $Volume -le 100) {
 			$iTunes.SoundVolume = $Volume
 			Write-Host "(PS iTunes): iTunes Volume" $iTunes.SoundVolume -BackgroundColor Black -ForegroundColor green	
 		}	
 		else {
 			Write-Host "(PS iTunes): Volume has to be 0-100" -BackgroundColor Black -ForegroundColor red
 		}
 	}
 }
 
 function Show-iTunesVersion { 
     process {
 	Write-Host "(PS iTunes): iTunes Version" $iTunes.version -BackgroundColor Black -ForegroundColor White	
 	}
 }
 
 function get-PSiTunesHelp { 
     process {
 	Write-Host " "
 	Write-Host " Powershell iTunes Help"
 	Write-Host " "	
 	Write-Host "********************************************************"	
 	Write-Host " "
 	Write-Host " Commands"
 	Write-Host " "
 	Write-Host " Start-iTunes			Starts iTunes"
 	Write-Host " Close-iTunes			Closes iTunes"
 	Write-Host " Play-Track			Plays a iTunes Track"
 	Write-Host " Pause-Track			Pauses a iTunes Track"
 	Write-Host " Resume-Track			Resumes a iTunes Track"
 	Write-Host " Back-Track			Go to the last iTunes Track"
 	Write-Host " Next-Track			Plays next iTunes Track"
 	Write-Host " Mute-Track			Mutes iTunes"
 	Write-Host " Unmute-Track			Unmutes iTunes"	
 	Write-Host " Show-iTunesVersion		Shows iTunes Version"
 	Write-Host " Set-VolumeDown			Set Volume Down"
 	Write-Host " Set-VolumeUp			Set Volume Up"
 	Write-Host " Set-Volume			Set iTunes Volume (0-100)"
 	}
 }
`

