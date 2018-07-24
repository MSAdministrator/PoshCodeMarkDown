---
Author: kris cieslak
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6476
Published Date: 2016-08-17t08
Archived Date: 2016-08-21t07
---

# get-kbinfo - 

## Description

identifying knowledge base article by its id number taken from string or filename.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 #
  PARAM ($filename = $(throw "Specifiy the file name"))
 
  $page = New-Object System.Net.WebClient;  
  $kb = [regex]::match($filename,'KB\d*|kb\d*').ToString();
  $p = $page.DownloadString('http://support.microsoft.com/kb/'+$kb)
  $p = [regex]::replace( [regex]::match($p,'<h1 class="title">.*</h1>').ToString(), '<h1 class="title">|</h1>', '' )
  write-host -Fore Yellow `n'[*] Filename: '$filename  
  write-host `n'    '$p `n'    ( http://support.microsoft.com/kb/'$kb' )'`n
`

