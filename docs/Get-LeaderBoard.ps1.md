---
Author: scott alvarino
Publisher: 
Copyright: 
Email: 
Version: 1.01
Encoding: ascii
License: cc0
PoshCode ID: 2632
Published Date: 2011-04-23t21
Archived Date: 2011-04-25t16
---

# get-leaderboard - 

## Description

script to retrieve the leader boards from the 2011 scripting games as objects.  new version fixes an error with regex that caused usernames with periods or other non-english characters to be excluded (thanks to scott alvarino for noticing this).  also added rankings, which unfortunately causes powershell to default the display to a list format, but i assume that any powersheller looking at the leaderboards is capable of piping the output to “format-table -autosize”

## Comments



## Usage



## TODO



## script

`get-leaderboard`

## Code

`#
 #
 <#
 	.SYNOPSIS
 		Pulls down the leaderboards for the 2011 Scripting Games
 
 	.DESCRIPTION
 		Quick and dirty script to pull down the leaderboards for the 2011 scripting games.  
 		Can choose either beginner or advanced via a command line switch. To see the output
 		in a table, you must pipe to "format-table -autosize" or "out-gridview"
 		
 	.PARAMETER  Level
 		The leaderboard to parse
 
 	.EXAMPLE
 		Get-LeaderBoard -Level Beginner
 		
 		Retrieves the beginner leaderboard.
 
 	.EXAMPLE
 		Get-LeaderBoard -Level Advanced
 		
 		Retrieves the advanced leaderboard
 	
 	.EXAMPLE
 		Get-LeaderBoard Advanced | Format-Table -Autosize
 		
 		Retrieves the advanced leaderboard and returns values in a table
 	
 	.EXAMPLE
 		Get-LeaderBoard Advanced | Format-Table -Autosize
 		
 		Retrieves the advanced leaderboard and displays a table.
 	
 	.EXAMPLE
 		Get-LeaderBoard Advanced | where { $_.UserName -eq "My Name" }
 		
 		Retrieves status for user "My Name"	
 	.NOTES
 		NAME:      Get-LeaderBoard
  		VERSION:   1.01
  		AUTHOR:    Alex McFarland
  		LASTEDIT:  04/23/2011
 	 	-Added rankings to output.  This unfortunately requires use of ft - auto to get 
 		 decent looking output.
 		-Incorporated regex changes to to issues points out by Scott Alvarino. Was
 		 missing foreign language characters and periods.
 #>
 function Get-LeaderBoard 
 {
 	param(
 		[Parameter(
 			Position = 0,
 			Mandatory = $true,
 			HelpMessage = "Leaderboard to parse.  Advanced, or Beginner")]
 		[ValidateSet("Advanced","Beginner")]
 		[String]$Level="Advanced"
 	)
 	
 	$WebClient = New-Object System.Net.WebClient
 	$LeaderPage = $WebClient.DownloadString("http://2011sg.poshcode.org/Reports/TopUsers?filter=$Level") 
 	
 	$RegEx = '<a href="/Scripts/By/\d{1,3}">(?<UserName>[\w\s\S]*)</a>\s*</td>\s*<td>\s*(?<TotalPoints>\d{1,2}\.\d{1,2})\s*</td>\s*<td>\s*(?<ScriptsRated>\d*)'
 	
 	$i = 1
 	$LeaderPage -split "<tr>" | ForEach { 
 		$_ -match $RegEx | Out-Null
 		if (-not $Matches.UserName -eq "") 
 		{
 			New-Object PSObject -Property @{
 				"Ranking" = $i++
 				"UserName" = $Matches.UserName
 				"ScriptsRated" = $Matches.ScriptsRated
 				"TotalPoints" = $Matches.TotalPoints
 				"AverageRating" = if  ($Matches.ScriptsRated -gt 0) 
 				{
 					"{0:N2}" -f ($Matches.TotalPoints / $Matches.ScriptsRated)
 				}
 			}
 		}
 		if ($Matches) { $Matches.Clear() }
 		
 	} | select -Unique Ranking,UserName,ScriptsRated,AverageRating,TotalPoints
 }
`

