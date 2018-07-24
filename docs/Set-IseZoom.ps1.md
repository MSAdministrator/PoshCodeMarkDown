---
Author: mwjcomputing
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5863
Published Date: 2015-05-20t13
Archived Date: 2015-05-22t03
---

# set-isezoom - 

## Description

sets the ise zoom level to a range between 20 and 400.

## Comments



## Usage



## TODO



## function

`set-isezoom`

## Code

`#
 #
 function Set-IseZoom {
   param(
     [Parameter(Mandatory=$true)]
     [ValidateRange(20,400)]
     [int]$ZoomAmount
   )
   
   Write-Verbose 'Starting Set-IseZoom'
 
   Write-Verbose "Setting zoom percentage to $ZoomAmount"
   $psISE.Options.Zoom=$ZoomAmount
 
   if ($psISE.Options.Zoom -ne $ZoomAmount) {
     Write-Warning -Message "Unable to set ISE Zoom level to $ZoomAmount"
   }
 
   Write-Verbose 'Stopping IseZoom'
 
   
     .SYNOPSIS
         This function sets the zoom level in the PowerShell ISE.
     .DESCRIPTION
         This function uses the ISE object model to set the zoom level to the specified value.
     .EXAMPLE
         Set-IseZoom -ZoomAmount 150
     .NOTES
         Written by Matt Johnson (@mwjcomputing)
     .LINK
         http://www.mwjcomputing.com/
     .LINK
         https://technet.microsoft.com/en-us/library/dd819500.aspx
     #>
 
 }
`

