---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2122
Published Date: 2011-09-07t09
Archived Date: 2016-06-29t20
---

# publish-file - 

## Description

use this to upload one or more files to a sharepoint document library. should also work with any webdav service, although that hasn’t been tested. the filename parameter expects fileinfo objects. easiest way to do so is to pass them on the pipeline from get-childitem.  this script is a refinement of a technique that i first saw here

## Comments



## Usage



## TODO



## function

`publish-file`

## Code

`#
 #
 function Publish-File {
 	param (
 		[parameter( Mandatory = $true, HelpMessage="URL pointing to a SharePoint document library (omit the '/forms/default.aspx' portion)." )]
 		[System.Uri]$Url,
 		[parameter( Mandatory = $true, ValueFromPipeline = $true, HelpMessage="One or more files to publish. Use 'dir' to produce correct object type." )]
 		[System.IO.FileInfo[]]$FileName,
 		[system.Management.Automation.PSCredential]$Credential
 	)
 	$wc = new-object System.Net.WebClient
 	if ( $Credential ) { $wc.Credentials = $Credential }
 	else { $wc.UseDefaultCredentials = $true }
 	$FileName | ForEach-Object {
 		$DestUrl = "{0}{1}{2}" -f $Url.ToString().TrimEnd("/"), "/", $_.Name
 		Write-Verbose "$( get-date -f s ): Uploading file: $_"
 		$wc.UploadFile( $DestUrl , "PUT", $_.FullName )
 		Write-Verbose "$( get-date -f s ): Upload completed"
 	}
 	
 }
 
`

