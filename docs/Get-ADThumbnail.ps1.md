---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5357
Published Date: 2015-08-05t14
Archived Date: 2015-03-24t01
---

# get-adthumbnail - 

## Description

gets the picture stored in the thumbnailphoto attribute for the specified user/users and saves it to the path specified by the path parameter. path should be a folder, not a file.

## Comments



## Usage



## TODO



## function

`get-adthumbnail`

## Code

`#
 #
 function Get-ADThumbnail
 {
     <#
     .SYNOPSIS
     Gets the picture stored in the thumbnailPhoto attribute for the specified user/users.
 
     .DESCRIPTION
     Gets the picture stored in the thumbnailPhoto attribute for the specified user/users and saves it to the path specified by the Path parameter. Path should be a folder, not a file.
 
     Requires the ActiveDirectory-module to run. Import it manually if you are not running PoSh v.3 or newer.
 
     .EXAMPLE
     Get-ADThumbnail -Identity MySamAccountName -Path C:\Temp\
 
     Downloads the picture of the user with SamAccountName 'MySamAccountName' and saves it in the folder C:\Temp\
 
     .EXAMPLE
     Get-ADUser -Filter "GivenName -eq 'John'" | Get-ADThumbnail -Path C:\Temp\
 
     Downloads the picture from all the users with the GivenName 'John' and saves the files to C:\Temp\
 
     .PARAMETER Identity
     Specify the SamAccountName, DistinguishedName, objectGUID or SID of the user(s) with the picture. Supports pipeline input.
 
     .PARAMETER Path
     Specify the folder were the picture(s) will be saved. The filename will be "SamAccountName.jpg"
 
     If omitted, this will default to the current path.
 
     #>
 
     [cmdletbinding()]
     param([Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
           [Alias('SamAccountName','DistinguishedName','ObjectGUID','SID')]
           $Identity,
           $Path = '.\')
 
     BEGIN { }
 
     PROCESS {
         
         if ((Test-Path $Path -PathType Leaf)) {
             Write-Error "Path should be a folder, not a file."
             return
         }
 
         foreach ($Ident in $Identity) {
 
             try {
                 $ADUser = Get-ADUser -Identity $Ident -Properties Thumbnailphoto -ErrorAction Stop
             }
             catch {
                 Write-Error "Failed to get the user $Ident from Active Directory."
                 Continue
             }
 
             [byte[]] $PictureByteArray = $ADUser | select -ExpandProperty Thumbnailphoto
 
             if ($PictureByteArray -eq $null) {
                 Write-Warning "No thumbnail found for user $Ident."
                 Continue
             }
 
             $PictureFileContent = [System.Text.Encoding]::Default.GetString($PictureByteArray)
 
             $FileName = "$($ADUser.SamAccountName).jpg"
 
             $FullPath =  Join-Path $Path $FileName
 
 
             Set-Content -Path $FullPath -Value $PictureFileContent
 
             Remove-Variable PictureByteArray, PictureFileContent, Filename -ErrorAction SilentlyContinue
         }
     }
 
     END { }
 }
`

