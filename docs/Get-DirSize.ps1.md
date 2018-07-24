---
Author: tojo2000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5387
Published Date: 2016-08-29t09
Archived Date: 2016-07-15t09
---

# get-dirsize - 

## Description

a v2.0 function to recursively get the sizes of all subdirectories under a root path.

## Comments



## Usage



## TODO



## function

`get-dirsize`

## Code

`#
 #
 function Get-DirSize {
 <#
 .Synopsis
   Gets a list of directories and sizes.
 .Description
   This function recursively walks the directory tree and returns the size of 
   each directory found.
 .Parameter path
   The path of the root folder to start scanning.
 .Example
   PS> (Get-DirSize $env:userprofile | sort Size)[-1]
 .Example
 	PS> $folders | Get-DirSize
 .Example
 	PS> 'Folder', 'Size' -join "`t" > folder_sizes.txt
 	PS> Get-DirSize C:\MyFolder | Sort-Object Size -desc | 
 	>>    %{$_.Folder, $_.Size -join "`t"} >> folder_sizes.txt 
 .ReturnValue
   An object with Folder and Size properties.
 #>
   param([Parameter(Mandatory = $true, ValueFromPipeline = $true)][string]$path)
   BEGIN {}
  
   PROCESS{
     $size = 0
     $folders = @()
   
     foreach ($file in (Get-ChildItem $path -Force -ea SilentlyContinue)) {
       if ($file.PSIsContainer) {
         $subfolders = @(Get-DirSize $file.FullName)
         $size += $subfolders[-1].Size
         $folders += $subfolders
       } else {
         $size += $file.Length
       }
     }
   
     $object = New-Object -TypeName PSObject
     $object | Add-Member -MemberType NoteProperty -Name Folder `
                          -Value (Get-Item $path).FullName
     $object | Add-Member -MemberType NoteProperty -Name Size -Value $size
     $folders += $object
     Write-Output $folders
   }
   
   END {}
 }
`

