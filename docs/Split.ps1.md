---
Author: tojo2000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2993
Published Date: 2012-10-08t10
Archived Date: 2012-02-05t05
---

# split - 

## Description

split a file into smaller files.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function Split {
 <#
 .Synopsis
 Splits up a file into smaller files.
 .Description
 This function takes a file as input and splits it into files by a set number of
 lines.
 .Parameter filename
 The name of the file to be used as an input.
 This value can be piped to the function (see examples)
 .Parameter lines
 The number of lines per file.
 .Parameter prefix
 A prefix for the output filenames.  Defaults to 'split.'
 .Parameter extension
 The file extension to use for output files.
 .Example
 Split -filename mybigfile.txt -lines 100
 .Example
 ls | ?{-not $_.PSIsContainer} | %{$_.FullName} | Split -lines 100
 #>
   param([Parameter(Mandatory=$true,
                    ValueFromPipeLine=$true)]
         [string]$filename,
         [Parameter(Mandatory=$true)]
         [int]$lines,
         [Parameter(Mandatory=$false)]
         [string]$prefix = 'split.',
         [Parameter(Mandatory=$false)]
         [string]$extension = 'txt')
         
   if (-not (Test-Path $file)) {
     Write-Host "$file not found!"
     break
   }
   
   $increment = 0
   $line = 0
   
   Get-Content $file |
     %{
       $line += 1
       
       if (-not ($lines % $line)) {
         $increment += 1
       }
       
       $_ >> "$file.$increment.$extension"
     }  
 }
`

