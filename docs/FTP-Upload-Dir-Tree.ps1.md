---
Author: lance robinson
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1140
Published Date: 2010-06-01t09
Archived Date: 2017-03-23t21
---

# ftp upload dir tree - 

## Description

uploads a directory tree to a remote ftp server.  uses netcmdlets (send-ftp).

## Comments



## Usage



## TODO



## function

`upload-directory`

## Code

`#
 #
 function upload-directory {
   param( [string] $server = $( Throw "You must specify an FTP server to logon to."),
 	 [string] $dir = $( Throw "You must specify a local directory to upload (ie, C:\Testing\FTPTest\)"),
 	 [switch] $overwrite = $false,
 	 [System.Management.Automation.PSCredential] $cred = $( Throw "You must provide credentials with which to logon to the FTP server.") ) 
         
   $files = (get-childitem $dir -r)
 
   foreach ($file in $files) {
     $remfilename = $file.FullName.Replace($dir, "")
     $remfilename = $remfilename.Replace("\", "/")
     if ($file.Attributes.ToString().IndexOf("Directory") -ge 0) {
   	  try
   	  {
       send-ftp -server $server -cred $cred -create $remfilename -overwrite:$overwrite
       }
     }
     else {
       send-ftp -server $server -cred $cred -localfile $file.FullName -remotefile $remfilename -overwrite:$overwrite
 
 }
`

