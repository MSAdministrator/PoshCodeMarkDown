---
Author: stefan stranger
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 910
Published Date: 2009-03-07t09
Archived Date: 2014-12-29t11
---

# twitbrain cheat - 

## Description

#twitbrain cheat powershell script

## Comments



## Usage



## TODO



## script

`publish-tweet`

## Code

`#
 #
 
 
 [System.Reflection.Assembly]::LoadWithPartialName(�System.Web") | Out-Null
 
 Function Publish-Tweet([string] $TweetText)
 { 
 	[System.Net.ServicePointManager]::Expect100Continue = $false
 	$request = [System.Net.WebRequest]::Create("http://twitter.com/statuses/update.xml")
 	$Username = "username"
 	$Password = "password"
 	$request.Credentials = new-object System.Net.NetworkCredential($Username, $Password)
 	$request.Method = "POST"
 	$request.ContentType = "application/x-www-form-urlencoded" 
 	write-progress "Tweeting" "Posting status update" -cu $tweetText
 	
 	$formdata = [System.Text.Encoding]::UTF8.GetBytes( "status="  + $tweetText  )
 	$requestStream = $request.GetRequestStream()
 		$requestStream.Write($formdata, 0, $formdata.Length)
 	$requestStream.Close()
 	$response = $request.GetResponse()
 	
 	write-host $response.statuscode 
 	$reader = new-object System.IO.StreamReader($response.GetResponseStream())
 		$reader.ReadToEnd()
 	$reader.Close()
 }
 
 Function Waiting()
 {
 	for ($a=15; $a -gt 1; $a--) 
 	{
 	Write-Progress -Activity "Waiting for next poll" `
 	-SecondsRemaining $a -Status "Please wait."
 	Start-Sleep 1
 	}
 }
 
 Write-Host "You are going to cheat;-)"
 $strResponse = Read-Host "Are you sure you want to continue? (Y/N)"
 
 if ($strResponse -eq 'N')
 	{
 	Write-host "Maybe a good choice. It has to be a fair competition ;-)"
 	break
 	}
 
 for (;;)
 {
     Write-host "Get calculation from Twitbrain website"
 	$ws = New-Object net.WebClient
 	
 	$html = $ws.DownloadString("http://ajaxorized.com/twitbrain")
 	
 	$currentdir = [Environment]::CurrentDirectory=(Get-Location -PSProvider FileSystem).ProviderPath
 	$html | Set-Content "$currentdir\Twitbrain.html"
 
 	$twitbrainpage = Get-Content "$currentdir\Twitbrain.html" | out-string
 	
 	$calc = [regex]::match($twitbrainpage,'(?<=\<div class="challenge"\>).+(?=\</div>
 					<p class="challenge-answer">)',"singleline").value.trim()
 	
 	$calc = $calc -replace "\*times\*","*"
 	$calc = $calc -replace "\+plus\+","+"
 	$calc = $calc -replace "\-minus\-","-"
 	
 	$result = invoke-expression $calc
 	
 	$tweet = "@twitbrain " + $result
 
 	$oldresult = Get-Content "$currentdir\oldresult.txt"
 	if ($result -eq $oldresult)
 	{
 		write-host "No new Twitbrain question is published yet"
 	}
 	else 
 	{
 		Write-host "What is the result of the next question?"
 		Write-host $calc
 		Publish-Tweet $tweet
 		write-host "Tweet publised"
 		$oldresult = $result
 		Write-host "Save oldresult to text file"
 		$oldresult > "$currentdir\oldresult.txt"
 	}
 
 	Waiting
 }
`

