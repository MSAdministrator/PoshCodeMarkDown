---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3636
Published Date: 2012-09-12t06
Archived Date: 2012-09-15t04
---

# start-timer - 

## Description

the kitchen timer with a kitchen sink -full of features

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 A syntax error on your $Status array definition - you forgot the braces around the elements;
 
 [string[]]$status=("Cooking","Burning")
 
 cheers, yeatsie
 
 
 ####################################################################################################
 param( $seconds=0,  $reason="The Timer",  $SoundFile="$env:SystemRoot\Media\notify.wav",
       $minutes=0,  $hours=0,  $timeout=0,  [switch]$novoice, [string[]]$status="Cooking","Burning"
 )
 
 $start = [DateTime]::Now
 write-progress -activity $reason -status "Running"
 
 $seconds = [Math]::Abs($seconds) + [Math]::Abs($minutes*60) + [Math]::Abs($hours*60*60)
 $end = $start.AddSeconds( $seconds )
 
 $Voice = new-object -com SAPI.SpVoice
 $Sound = new-Object System.Media.SoundPlayer
 
 if(Test-Path $soundFile) {
    $Sound.SoundLocation=$SoundFile
 } else {
    $soundFile = $Null
 }
 
 do {
    $remain  = ($end - [DateTime]::Now).TotalSeconds
    $percent = [Math]::Min(100, ((($seconds - $remain)/$seconds) * 100))
    $sec     = [Math]::Max(000, $remain)
 
    write-progress -activity $reason -status $status[0] -percent $percent -seconds $sec
    start-sleep -milli 100
 } while( $remain -gt 0 )
 
 $Host.UI.RawUI.FlushInputBuffer()
 do {
    if($SoundFile) {
       $Sound.PlaySync()
    } else {
       [System.Media.SystemSounds]::Exclamation.Play()
    }
    if(!$novoice -and !$Host.UI.RawUI.KeyAvailable){
      $null = $Voice.Speak( "$reason is $($status[1])!!", 0 )
    }
 
    write-progress -activity $reason -status $status[1] -percent ([Math]::Min(100,((($seconds - $remain)/$seconds) * 100))) -seconds ([Math]::Max(0,$remain))
    $remain = ($end - [DateTime]::Now).TotalSeconds
 } while( !$Host.UI.RawUI.KeyAvailable -and ($timeout -eq 0 -or $remain -gt -$timeout))
 
 if($SoundFile) {
    $Sound.Stop() 
 }
`

