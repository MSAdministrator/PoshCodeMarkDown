---
Author: bernd kriszio
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5890
Published Date: 2017-06-10t15
Archived Date: 2017-05-22t18
---

# set-defaultbrowser - 

## Description

with this function i can switch between ie and firefox as default browser. it is just a start. there might be some more registry keys to be changed

## Comments



## Usage



## TODO



## function

`set-defaultbrowser`

## Code

`#
 #
 function Set-defaultBrowser
 {
     param($defaultBrowser)
 
     switch ($defaultBrowser)
     {
         'IE' {
             Set-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\ftp\UserChoice' -name ProgId IE.FTP
             Set-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\http\UserChoice' -name ProgId IE.HTTP
             Set-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\https\UserChoice' -name ProgId IE.HTTPS
             
         }
         'FF' {
             Set-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\ftp\UserChoice' -name ProgId FirefoxURL
             Set-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\http\UserChoice' -name ProgId FirefoxURL
             Set-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\https\UserChoice' -name ProgId FirefoxURL
         }
 	'Chrome' {
 	    Set-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\ftp\UserChoice' -name ProgId ChromeHTML
             Set-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\http\UserChoice' -name ProgId ChromeHTML
             Set-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\https\UserChoice' -name ProgId ChromeHTML
         }
     } 
     
         
 <#
 (Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\ftp\UserChoice').ProgId
 (Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\http\UserChoice').ProgId
 (Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\Shell\Associations\UrlAssociations\https\UserChoice').ProgId
 #>
 
 }
 
`

