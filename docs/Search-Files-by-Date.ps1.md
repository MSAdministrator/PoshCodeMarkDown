---
Author: matt schmitt
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3806
Published Date: 2012-11-30t09
Archived Date: 2012-12-04t12
---

# search files by date - 

## Description

please

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
   Author:   Matt Schmitt
   Date:     11/30/12 
   Version:  1.0 
   From:     USA 
   Email:    ithink2020@gmail.com 
   Website:  http://about.me/schmittmatt
   Twitter:  @MatthewASchmitt
   
   Description
   A script for finding files in a directory by Last Accessed Date.  
 #>
 
 
 Write-Host "Enter Root Directory you would like to search"
 Write-Host ""
 Write-Host "Example: C:\Users\testuser"
 Write-Host ""
 $directory = Read-Host ">>"
 
 cls
 Write-Host "Enter "From" Date"
 Write-Host ""
 Write-Host "Example: 1/1/12"
 Write-Host ""
 $startDate = Read-Host '>>'
 
 cls
 Write-Host "Enter "To" Date"
 Write-Host ""
 Write-Host "Example: 11/30/12"
 Write-Host ""
 $endDate = Read-Host '>>'
 
 cls
 Write-Host "Enter where you would like the CSV file to be exported"
 Write-Host ""
 Write-Host "Example: C:\Files.csv"
 Write-Host ""
 $outPutFile = Read-Host '>>'
 
 cls
 Write-Host ""
 Write-Host ""
 
 Write-Host "Searching for files in '$directory' from '$startDate' to '$endDate'..."
 
 Get-ChildItem -path $directory -Recurse  | where { $_.lastaccesstime -ge [datetime]$startDate -and $_.lastaccesstime -lt [datetime]$endDate} | select fullname | Export-CSV -Path $outPutFile
 
 Write-Host ""
 Write-Host "Results have been exported!"
 Write-Host ""
 Write-Host "Press any key to Exit."
 
 $x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
`

