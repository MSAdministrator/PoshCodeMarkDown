---
Author: sunny chakrabort
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 4628
Published Date: 2013-11-20t12
Archived Date: 2013-11-25t06
---

# make a phone call - 

## Description

make a phone call from powershell and pass texttospeech variables like computername, diskfreespace.

## Comments



## Usage



## TODO



## script

`get-freediskpercentforc`

## Code

`#
 #
 <#
 
 .NOTES
     AUTHOR: Sunny Chakraborty(sunnyc7@gmail.com)
 	WEBSITE: http://tekout.wordpress.com
     VERSION: 0.1
 	CREATED: 17th April, 2012
 	LASTEDIT: 17th April, 2012
 	Requires: PowerShell v2 or better
 
 .CHANGELOG
 4/17/2012 Try passing powershell objects to PROTO API and pass the variables to .JS file
 	Pass other system variables and check if text to speech can translate double or a double-to-char conversion is required.
 
 .SYNOPSIS
     Make a phone call from Powershell.
 	
 .DESCRIPTION
 	The script demonstrates how you can collect state-data in powershell and pass it as an argument to a REST API call and alert a System Admin.
 	For this example, TROPO REST API's were used. (www.tropo.com)
 	The phone-number will receive a Call with the following text To speech
 		Please check the server $this. 
 		The percent Free Space on C Drive is $inDecimals.
 
 	This is a proof of concept. V 0.1
 	There are numerous areas of improvement. 
 	
 .IMPORTANT
 	Please create a new account and setup your application in tropo. Its free for dev use. http://www.tropo.com
 	Copy and replace the TOKEN in your application with the TOKEN below to initiate a call.
 	
 .OTHER
 
 JAVASCRIPT (Hosted on Tropo)
 
 TropoTest.js
 call('+' + numToCall , {
   timeout:30,
   callerID:'19172688401',
       onAnswer: function() {
       	say("Houston ! We have a problem ");
 	say("Please check the server" + sourceServer );
 	say("The percent Free Space on C Drive is" + freeCDisk );
 	say("Goodbye.");
 	log("Call logged Successfully");
   },
   onTimeout: function() {
       	log("Call timed out");
   },
   onCallFailure: function() {
       	log("Call could not be completed as dialed");
   }
 });
 
 #>
 
 $baseUrl = "https://api.tropo.com/1.0/sessions?action=create&"
 
 $protoToken = "10b0026696a79f448eb21d8dbc69d78acf12e2f1f62f291feecec8f2b8d1eac76da63d91dd317061a5a9eeb0"
 $myCell = '1234567890'
 
 Function get-FreeDiskPercentForC {
 $disk = Get-DiskUsage
 $free = $disk[0].FreeSpace / $disk[0].TotalSize
 $freeDiskCPercent = [System.Math]::Round($free, 2)
 $freeDiskCPercent
 }
 
 $sourceServer =hostname
 $cDisk = get-FreeDiskPercentForC
 
 $callThis = $baseUrl+ 'token=' + $protoToken + '&numToCall=' + $myCell + '&sourceServer=' + $sourceServer + '&freeCDisk=' + $cDisk
 
 $newClient = new-object System.Net.WebClient
 $newClient.DownloadString($callThis)
`

