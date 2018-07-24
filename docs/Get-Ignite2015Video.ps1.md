---
Author: mwjcomputing
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5857
Published Date: 2015-05-12t13
Archived Date: 2015-05-16t04
---

# get-ignite2015video - 

## Description

this function downloads sessions from microsoft ignite 2015.

## Comments



## Usage



## TODO



## function

`get-ignite2015video`

## Code

`#
 #
 function Get-Ignite2015Video {
   [CmdletBinding()]
   param (
     [Parameter(Mandatory=$True,ValueFromPipeline=$true)]
     [string]$Session,
     [string]$Path = "C:\ignite\2015-videos"
   )
 
   begin {
     $fileUrl = "http://video.ch9.ms/sessions/ignite/2015/$($Session.ToUpper()).mp4"
 
     Write-Verbose -Message "URL: $fileUrl"
     
     if(!(Test-Path $path)) {
       New-Item $path -ItemType directory -Force
     }
 
     Import-Module -Name 'BitsTransfer'
   }
 
   process {
     Start-BitsTransfer -Source $fileUrl -Destination "$path\$Session.wmv" -Description "Downloading file $Session from Ignite 2015."
   }
 
   end {
     Remove-Module -Name 'BitsTransfer'
   }
 }
`

