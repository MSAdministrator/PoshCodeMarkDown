---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5075
Published Date: 2014-04-13t07
Archived Date: 2014-04-15t23
---

# get-jpegdata.ps1 - 

## Description

reads jpeg metadata

## Comments



## Usage



## TODO



## function

`get-jpegdata`

## Code

`#
 #
 requires -version 2.0
 function Get-JpegData {
   param(
     [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$JpegFile
   )
   
   begin {
     $asm = [AppDomain]::CurrentDomain.GetAssemblies()
     if (!($asm | ? {
       $_.Fullname.Contains('PresentationCore')
     })) {
       $asm = (gci -r ([Regex]"(?<=file:///)(.*)(?=GAC.*)").Match(
         ($asm[0].Evidence | ? {$_ -match 'file:'}).Value
       ).Value | ? {$_.Name -eq 'PresentationCore.dll'}).FullName
       
       [void][Reflection.Assembly]::LoadFile($asm)
     }
     
     ($asm | ? {
       $_.Fullname.Contains('PresentationCore')
     }).GetType(
       'System.Windows.Media.Imaging.BitmapMetadata'
     ).GetProperties() | % {$chk = @()}{
       if ($_.Name -ne 'DependencyObjectType') { $chk += $_.Name }
     }
     
     $JpegFile = cvpa $JpegFile
   }
   process {
     try {
       $fs = [IO.File]::OpenRead($JpegFile)
       
       $dec = New-Object Windows.Media.Imaging.JpegBitmapDecoder(
         $fs, [Windows.Media.Imaging.BitmapCreateOptions]::IgnoreColorProfile,
         [Windows.Media.Imaging.BitmapCacheOption]::Default
       )
       $chk | % {$raw = $dec.Frames[0].Metadata}{
         if ($_ -eq 'Author') { '{0, 19}: {1}' -f $_, $raw.$_[0] }
         else { '{0, 19}: {1}' -f $_, $raw.$_ }
       }
     }
     catch {
       Write-Host $_.Exception.Message -fo Magenta
     }
     finally {
       if ($fs -ne  $null) { $fs.Close() }
     }
   }
   end {}
 }
`

