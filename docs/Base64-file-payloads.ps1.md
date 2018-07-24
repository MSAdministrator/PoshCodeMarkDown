---
Author: nicolas tremblay
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4296
Published Date: 2013-07-07t16
Archived Date: 2016-10-27t14
---

# base64 file payloads - 

## Description

create/insert/extract well formatted and indexed based64 payload of files

## Comments



## Usage



## TODO



## function

`encode-filetobase64block`

## Code

`#
 #
 function Encode-FileToBase64Block {
     param ( [ValidateScript({Test-Path $_ -PathType 'Leaf'})][string]$SourcePath ) 
     $leafName = (Get-Item $SourcePath).Name 
     [system.convert]::ToBase64String((Get-Content $SourcePath -Encoding Byte), "InsertLineBreaks") 
 }
 
 
 function Get-Base64BlockSection { 
     param ( 
             [ValidateScript({Test-Path $_ -PathType 'Leaf'})][string]$SourcePath,
             [switch]$List, 
             [string]$Indexfile
           )
     $outputFilenames   = $encodedStartLines | % { $_.Line.Split("|")[1]}
     if ($outputFilenames.Count -gt 0)
     {
         $metadataBlock = @{}
         foreach ($item in $outputFilenames)
         {
             $metadataBlock[$item] = @{}
         }
         if ($List.IsPresent) { return $metadataBlock.Keys | sort }
         if (!$Indexfile) 
         { 
             Write-Host "ERROR: No index specified for the '-Indexfile' parameter" -ForegroundColor Red
             Write-Host "Available indexe(s) found are: $($outputFilenames -join " , ")`n`n"
             throw "No index specified"
         }
         else
         {
             $extractor = $metadataBlock.Keys | where { $_ -eq $Indexfile}
             if ($extractor)
             {
                 (Get-Content $SourcePath)[$($metadataBlock[$extractor].StartLine) .. $($metadataBlock[$extractor].EndLine - 2 )]
             }
             else
             {
                 Write-Host "ERROR: No Base64 Block index '$Indexfile' was found in file: $SourcePath`n`n" -ForegroundColor Red
                 throw "Invalid index"
             }
         }
     }
     else
     {
         if ($List.IsPresent) {return $null}
         Write-Host "ERROR: No Base64 Block indexes detected in file: ${SourcePath}`n`n" -ForegroundColor Red
         throw "No block index detected"
     }
 }
 
 function Invoke-base64Extraction {
     param ( 
             [ValidateScript({Test-Path $_ -PathType 'Leaf'})][string]$SourcePath,
             [switch]$All, 
             [string]$Indexfile
           )
     if ($All.IsPresent) 
     {
         $fileList =  Get-Base64BlockSection -SourcePath $SourcePath -List
         $fileList | % { Invoke-base64Extraction -SourcePath $SourcePath -Indexfile $_ }
         return
     }
     Split-Path -Parent $MyInvocation.InvocationName
     $outputContentPath = Join-Path $defaultpath $Indexfile
     [system.convert]::FromBase64String((Get-Base64BlockSection -SourcePath $SourcePath -Indexfile $Indexfile)) | Set-Content $outputContentPath -Encoding Byte
 }
 
 
 
 
 
`

