---
Author: stefan stranger
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1836
Published Date: 
Archived Date: 2010-11-10t02
---

# publish friendfeed entry - 

## Description

#publish ff entry using powershell script

## Comments



## Usage



## TODO



## script

`publish-ffentry`

## Code

`#
 #
 
 [System.Reflection.Assembly]::LoadWithPartialName(�System.Web") | Out-Null
 
 Function Publish-FFEntry([string] $FFEntryText)
 { 
 	[System.Net.ServicePointManager]::Expect100Continue = $false
 	$request = [System.Net.WebRequest]::Create("http://friendfeed-api.com/v2/entry")
 	$Username = "username"
 	$Remotekey = "remotekey"
 	$request.Credentials = new-object System.Net.NetworkCredential($Username, $Remotekey)
 	$request.Method = "POST"
 	$request.ContentType = "application/x-www-form-urlencoded" 
 	write-progress "Publishing FF Entry" "Posting Entry" -cu $FFEntryText
 	
 	$formdata = [System.Text.Encoding]::UTF8.GetBytes( "body="  + $FFEntryText  )
 	$requestStream = $request.GetRequestStream()
 		$requestStream.Write($formdata, 0, $formdata.Length)
 	$requestStream.Close()
 	$response = $request.GetResponse()
 	
 	write-host $response.statuscode 
 	$reader = new-object System.IO.StreamReader($response.GetResponseStream())
 		$reader.ReadToEnd()
 	$reader.Close()
 }
 
 [System.Reflection.Assembly]::LoadWithPartialName(�System.Web") | Out-Null
 
 Function Publish-FFEntry([string] $FFEntryText)
 { 
 	[System.Net.ServicePointManager]::Expect100Continue = $false
 	$request = [System.Net.WebRequest]::Create("http://friendfeed-api.com/v2/entry")
 	$Username = "username"
 	$Remotekey = "remotekey"
 	$request.Credentials = new-object System.Net.NetworkCredential($Username, $Remotekey)
 	$request.Method = "POST"
 	$request.ContentType = "application/x-www-form-urlencoded" 
 	write-progress "Publishing FF Entry" "Posting Entry" -cu $FFEntryText
 	
 	$formdata = [System.Text.Encoding]::UTF8.GetBytes( "body="  + $FFEntryText  )
 	$requestStream = $request.GetRequestStream()
 		$requestStream.Write($formdata, 0, $formdata.Length)
 	$requestStream.Close()
 	$response = $request.GetResponse()
 	
 	write-host $response.statuscode 
 	$reader = new-object System.IO.StreamReader($response.GetResponseStream())
 		$reader.ReadToEnd()
 	$reader.Close()
 }
 
 Publish-FFentry "This entry is being published using a PowerShell script. Yabbadabadoo!!"
`

