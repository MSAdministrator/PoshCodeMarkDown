---
Author: simon64
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5968
Published Date: 2016-08-07t20
Archived Date: 2016-05-17t12
---

# zipfile module - 

## Description

new-zipfile and expand-zipfile — two functions for zipping and unzipping the way i like doing it…

## Comments



## Usage



## TODO



## function

`new-zipfile`

## Code

`#
 #
 
 function New-ZipFile {
     #.Synopsis
     [CmdletBinding()]
     param(
     [Parameter(Position=0, Mandatory=$true)]
     $ZipFilePath,
     
     [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
     [Alias("PSPath","Item")]
     [array]$InputObject = $Pwd,
     
     [Switch]$Append,
     
     [System.IO.Compression.CompressionLevel]$Compression = "Optimal"
     )
     begin {
         [string]$File = Split-Path $ZipFilePath -Leaf
         [string]$Folder = $(if($Folder = Split-Path $ZipFilePath) { Resolve-Path $Folder } else { $Pwd })
         $ZipFilePath = Join-Path $Folder $File
         if(!$Append) {
         if(Test-Path $ZipFilePath) { Remove-Item $ZipFilePath }
             }
         $Archive = [System.IO.Compression.ZipFile]::Open( $ZipFilePath, "Update" )
         $inputobject = $InputObject|?{$_.PSIsContainer -eq $false}
         }
     process {
         foreach($path in $InputObject) {
             foreach($item in Resolve-Path $path.fullname) {
                 Push-Location (Split-Path $item)
                 
                 $relative = ($path.fullname ).substring(3)
                 
                 $null = [System.IO.Compression.ZipFileExtensions]::CreateEntryFromFile($Archive, $path.fullname, $relative, $Compression)
                 
                 Pop-Location
                 
                 }
             }
         }
     end {
         $Archive.Dispose()
         Get-Item $ZipFilePath
         }
     }
`

