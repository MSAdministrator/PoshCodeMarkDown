---
Author: andy arismendi
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2784
Published Date: 2011-07-11t17
Archived Date: 2011-07-16t11
---

# add-sharepointlibraryfil - 

## Description

add-sharepointlibraryfile – uploads a file to a sharepoint library via http put.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Add-SharePointLibraryFile
 {
 	param
 	(
 		[System.IO.FileInfo] $File, 
 		[System.Uri] $DocumentLibraryUrl = $(throw "Parameter -DocumentLibraryUrl [System.Uri] is required.")
 	)
 
 	Begin
 	{ }
 	
 	Process
 	{
 		if($_)
 		{
 			$File = [System.IO.FileInfo] $_
 		}
 		
 		if (!$File.Exists)
 		{
 			Write-Error ($File.ToString() + " does not exist.")
 			return $null
 		}
 		
 		trap
 		{
 			Write-Error $_.Exception.Message
 			break
 		}
 	
 		[string]$webPath = $DocumentLibraryUrl
 		if (-not $webPath.EndsWith("/")) {
 			$webPath += "/";
 		}
 		$webPath += $File.Name
 	
 		$request = [System.Net.WebRequest]::Create($webPath)
 	
 		$request.Credentials = [System.Net.CredentialCache]::DefaultCredentials
 		$request.Method = "PUT";
 	
 		$fileBuffer = New-Object byte[] 1024
 	
 		$stream = $request.GetRequestStream()
 		
 		$fsWorkbook = $file.OpenRead()
 		
 		$startBuffer = $fsWorkbook.Read($fileBuffer, 0, $fileBuffer.Length);
 		
 		for ($i = $startBuffer; $i -gt 0; $i = $fsWorkbook.Read($fileBuffer, 0, $fileBuffer.Length))
 		{
 			$stream.Write($fileBuffer, 0, $i);
 		}
 		
 		$stream.Close()
 	
 		$response = $request.GetResponse();
 	
 		$response.Close();
 	}
 	
 	End
 	{ }
 }
 
 
 $url = "http://sharepoint/sites/andy/files"
 
 Get-Item 'C:\Documents and Settings\andy\Desktop\doc.pdf' | Add-SharePointLibraryFile -DocumentLibraryUrl $url
`

