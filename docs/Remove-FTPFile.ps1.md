---
Author: xandertrystin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3463
Published Date: 2013-06-20t04
Archived Date: 2016-06-06t19
---

# remove-ftpfile - 

## Description

just a short function i wrote based on ftp upload and download examples. requires and absolute path to the file on the ftp server that you wish to remove i.e.  ftp

## Comments



## Usage



## TODO



## function

`remove-ftpfile`

## Code

`#
 #
 function Remove-FTPFile ($Source,$UserName,$Password)
 {
   $ftprequest = [System.Net.FtpWebRequest]::Create($Source)
   
 	$ftprequest.Credentials = New-Object System.Net.NetworkCredential($username,$password)
 	
 	$ftprequest.Method = [System.Net.WebRequestMethods+Ftp]::DeleteFile
 	
 	$ftpresponse = $ftprequest.GetResponse()  
 	$ftpresponse
 	
 }
`

