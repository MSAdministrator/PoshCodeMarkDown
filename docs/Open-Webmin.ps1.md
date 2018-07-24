---
Author: kim shepherd
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5840
Published Date: 2015-05-01t15
Archived Date: 2015-05-02t13
---

# open webmin - 

## Description

allows you to open a webmin interface in ie from the command line, automatically skipping an ssl error and logging in with the provided username and password.  i use this script as an external tool in mremoteng with the following powershell arguments line

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 Param(  
     [string]$Username,
     [string]$Password,
     [String]$URL
 )
 
 $ie = New-Object -com internetexplorer.application 
 $ie.visible = $true
 
 $ie.navigate($URL)
 
 while ($ie.ReadyState -ne 4)
 {
     Start-Sleep -Seconds 1;
 }
 
 if ($ie.document.url.contains("SSLError"))
 {
     ($ie.Document.getElementsByTagName("a") | where { $_.innerText.toLower().Contains("continue to this website") } | select -first 1).click()
 
     Start-Sleep -Seconds 1;
 
     while ($ie.ReadyState -ne 4)
     {
         Start-Sleep -Seconds 1
     }
 }
 
 $ie.Document.getElementById("user").value = $Username
 $ie.Document.getElementByID("pass").value = $Password
 ($ie.Document.getElementsByTagName("input") | where {$_.Value -eq "Login"}).click()
`

