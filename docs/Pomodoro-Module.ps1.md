---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3078
Published Date: 2012-12-03t23
Archived Date: 2015-05-04t23
---

# pomodoro module - 

## Description

a pomodoro module in powershell allowing you to run pomodoros in powershell with sounds being played, and live update in the progress bar

## Comments



## Usage



## TODO



## module

`stop-pomodoro`

## Code

`#
 #
 
 
 
 
 $script:DefaultLength = 25;
 $script:DefaultBreak = 5;
 $script:timer = $null
 $script:showprogress = $true
 
 function Stop-Pomodoro
 {
 	[CmdletBinding()]
 	param(
 	[parameter()]
 	$reason
 	)
 	Unregister-Event "Pomodoro" -ErrorAction silentlycontinue	
 	$script:timer = $null
 }
 
 function Set-PomodoroPrompt
 {
  [CmdletBinding()]
  param ()
  $script:promptbackup = $function:prompt 
  $function:prompt = { Get-PomodoroStatus -ForPrompt}  
 }
 
 function Restore-PomodoroPrompt
 {
 [CmdletBinding()]
  param ()
  $function:prompt = $script:promptbackup   
 }
 function Show-PomodoroProgress
 {
  [cmdletbinding()]param()
  $script:showprogress = $true
 }
 function Hide-PomodoroProgress
 {
 [cmdletbinding()]param()
 $script:showprogress = $false
 }
 
 function Get-PomodoroStatus
 {
  [CmdletBinding(DefaultParameterSetName="summary")] 
  param(
   [Parameter(ParameterSetName="remaining",Position=0)] 
   [switch]$remaining
   ,
   [Parameter(ParameterSetName="From",Position=0)] 
   [switch]$From
   ,
   [Parameter(ParameterSetName="Until",Position=0)] 
   [switch]$Until
   ,
   [Parameter(ParameterSetName="Length",Position=0)] 
   [switch]$Length  
   ,
   [Parameter(ParameterSetName="forPrompt",Position=0)] 
   [switch]$ForPrompt  
  )
    
 	if($script:timer)
 	{
 		if($script:pomoorbreak) 
 		 	{ 
 				$prefix = "Pomodoro - $script:currentwork"			
 				$pomotime = new-object system.TimeSpan 0,0,$script:currentlength,0
 		 	} 
 		else 
 			{ 
 				$prefix = "Having a Break from work - $script:currentwork"
 				$pomotime = new-object system.TimeSpan 0,0,$script:currentbreak,0
 			}
 		 
 		$diff = (get-Date) - $script:starttime	 
 		$timeleft = $pomotime - $diff
 		$endtime = $starttime + $pomotime
 	}
 	
 	switch ($PsCmdlet.ParameterSetName) 
     { 
     "summary"  { "{5} for {4} minutes from {0:hh:mm} to {1:hh:mm} - {2}:{3:00} minutes left." -f $starttime,$endtime ,$timeleft.minutes, $timeleft.seconds	,$pomotime.minutes,$prefix} 
     "remaining"  { $timeleft} 
 	"From" {$script:starttime}
 	"Until" { $endtime }
 	"Length" {$pomotime}
 	"ForPrompt" {
 				if($script:timer)
 				{
 					if ($script:pomoorbreak)
 						{
 						 if($script:currentwork -and ($script:currentwork.trim() -ne [string]::Empty))
 						  {
 						    "{0} {1}:{2:00}>" -f $(if($script:currentwork.length -gt 8) { $script:currentwork.substring(0,8)} else {$script:currentwork} ),
 									$timeleft.minutes, $timeleft.seconds
 						  }
 						 else
 						  {
 						  "{0} {1}:{2:00}>" -f "Pomo",$timeleft.minutes, $timeleft.seconds
 						  }
 						}
 					else
 					    {
 						  "{0} {1}:{2:00}>" -f "Break",$timeleft.minutes, $timeleft.seconds
 						}
 				}
 				else
 				{
 				 "No Pomo>"
 				}
 				
 		} 
     } 
 	
 	
 
 }
 function Start-Pomodoro
 {
 	[CmdletBinding()]
 	param (
 	  [Parameter()]
 	  [int]$Length = $script:DefaultLength
 	  ,
 	  [Parameter()]
 	  [int]$Break = $script:DefaultBreak
 	  ,
 	  [Parameter()]
 	  [string]$Work
 	  ,
 	  [Parameter()]
 	  [switch]$ShowPercent
 	  ,
 	  [Parameter()]
 	  [switch]$HideProgress
 	  ,
 	  [Parameter()]
 	  [switch]$UsePowerShellPrompt
 	 )
 	$script:currentlength = $length
 	$script:currentbreak = $break;
 	$script:currentshowpercent = [bool]$showpercent;
 	$script:currentwork = $work
 	if($HideProgress) { $script:showprogress = $false } else { $script:showprogress = $true }
 	Unregister-Event "Pomodoro" -ErrorAction silentlycontinue
 	$script:timer = $null
 	
 	$script:pomoorbreak = $true
 	$script:starttime = get-Date
 	
    $script:timer = New-Object System.Timers.Timer 
    $script:timer.Interval = 1000 
    $script:timer.Enabled = $true 
    
    $null = Register-ObjectEvent $timer "Elapsed" -SourceIdentifier "Pomodoro"  -Action {    
 	 $breakmode = & (get-Module Pomodoro) { $script:PomoOrBreak }
 	 $starttime = & (get-Module Pomodoro) { $script:starttime }
 	 $break = & (get-Module Pomodoro) {$script:currentbreak}
 	 $length = & (get-Module Pomodoro) { $script:currentlength }	 	 	 
 	 $work= & (get-Module Pomodoro) { $script:currentwork }	 	 	 
 	 if($breakmode) 
 	 	{ 
 			$prefix = "Pomodoro - $work "			
 			$pomotime = new-object system.TimeSpan 0,0,$length,0
 	 	} 
 	else 
 		{ 
 			$prefix = "Having a Break from work - $work"
 			$pomotime = new-object system.TimeSpan 0,0,$break,0
 		}
 	 
 	 $diff = (get-Date) - $starttime
 	 
 	$timeleft = $pomotime - $diff
 	$endtime = $starttime + $pomotime
 	$timeleftdisplay  = "for {4} minutes from {0:hh:mm} to {1:hh:mm} - {2}:{3:00} minutes left." -f $starttime,$endtime ,$timeleft.minutes, $timeleft.seconds	,$pomotime.minutes
 		if (($pomotime - $diff) -le 0) 
 		  {      
 		     write-Progress -Activity " " -Status "done"; 
 			 $sound = new-Object System.Media.SoundPlayer;
 		 	if ($breakmode) 
 			   {
 			     $sound.SoundLocation="$env:systemroot\Media\tada.wav";
 			   }
 			 else
 			   {
 			     $sound.SoundLocation="$env:systemroot\Media\notify.wav";
 			   }		
 			 $sound.Play();
 			 sleep 1
 			 $sound.Play();
 			 sleep 1
 			 $sound.Play();
 			 iex "& (get-module pomodoro) {`$script:starttime = get-Date; `$script:pomoorbreak = ! `$$breakmode } "
 
 		  }
 		else 
 		  {	
 		     if ( & (get-Module Pomodoro) { $script:showprogress } )
 				{
 
 					 if (& (get-Module Pomodoro) { $script:currentshowpercent}  ) 
 					  {	 	
 						$perc =100 - ( [int] ([double]$timeleft.totalseconds * 100 / [double]$pomotime.totalseconds))
 						write-Progress -Activity $prefix -Status "$timeleftdisplay" -PercentComplete $perc
 					  }
 					 else
 					  {
 					   write-Progress -Activity $prefix -Status "$timeleftdisplay"
 					  }
 				}	  
 		  }
 
    }
    if($UsePowerShellPrompt) { Set-PomodoroPrompt }
   
   
 }
 
 export-ModuleMember -Function "*"
 $myInvocation.MyCommand.ScriptBlock.Module.OnRemove = {
     if ($script:promptbackup) { $function:prompt = $script:promptbackup }	
 }
`

