---
Author: govnyakha
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6130
Published Date: 2016-12-04t10
Archived Date: 2016-05-17t20
---

# doxc2txt wi'out msoffice - 

## Description

original post found at http

## Comments



## Usage



## TODO



## function

`convert-docx2text`

## Code

`#
 #
 function Convert-Docx2Text {
   <#
     .SYNOPSIS
         Converts DOCX to TXT file without MS Office.
     .DESCRIPTION
         Creates text file in the same directory where original document
         is placed. This demo function does not use third party tools or
         libraries.
     .EXAMPLE
         PS C:\Users\Admin\Documents> Convert-Docx2Text pecoff_v83.docx
   #>
   param(
     [Parameter(Mandatory=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$FileName
   )
   
   begin {
     $FindMimeFromData = {
       [OutputType([String])]
       param(
         [Parameter(Mandatory=$true)]
         [ValidateNotNullOrEmpty()]
         [String]$FileName
       )
       
       Add-Type -AssemblyName ($$ = 'PresentationCore')
       try {
         $fs = [IO.File]::OpenRead($FileName)
         if (($len = $fs.Length) -gt 4096) { $len = 4096 }
         $buf = New-Object "Byte[]" $len
         [void]$fs.Read($buf, 0, $buf.Length)
         
         $gch = [Runtime.InteropServices.GCHandle]::Alloc($buf, 'Pinned')
         $ptr = $gch.AddrOfPinnedObject()
         
         if (([AppDomain]::CurrentDomain.GetAssemblies() | ? {
           $_.ManifestModule.ScopeName.Equals("$$.dll")
         }).GetType(
           'MS.Win32.Compile.UnsafeNativeMethods'
         ).GetMethod(
           'FindMimeFromData', [Reflection.BindingFlags]40
         ).Invoke($null, (
           $$ = [Object[]]($null, $FileName, $ptr, [Int32]$len, $null, 0, $mime, 0)
         )) -ne 0) {
           throw New-Object Exception('Could not get MIME type.')
         }
       }
       catch { $_.Exception }
       finally {
         if ($gch -ne $null) { $gch.Free() }
         if ($fs -ne $null) { $fs.Close() }
       }
     }
     
     $FileName = Convert-Path $FileName
   }
   process {
     if (($ext = (Get-Item $FileName).Extension) -ne '.docx'-and
         $FindMimeFromData.Invoke($FileName) -ne 'application/x-zip-compressed'
     ) {
       Write-Warning 'unsupported file format.'
     }
     try {
       $fs = [IO.File]::OpenRead($FileName)
       $br = New-Object IO.BinaryReader($fs)
       
       while ($true) {
         $fs.Position += 14
         $fs.Position += 4
         if ((-join $br.ReadChars($fnl)) -eq 'word/document.xml') {
             $ds = New-Object IO.Compression.DeflateStream($fs, 'Decompress')
             $of = [IO.File]::Create(($$ = $FileName -replace $ext, '.txt'))
             
             while ($true) {
               $buf = New-Object "Byte[]" 100
               $get = $ds.Read($buf, 0, $buf.Length)
               $of.Write($buf, 0, $get)
               
               if ($get -ne $buf.Length) {break}
             }
           }
           catch { $_.Exception }
           finally {
             if ($of -ne $null) { $of.Close() }
             if ($ds -ne $null) { $ds.Close() }
           }
           break
         }
         $fs.Position += $efl + $csz
       }
     }
     catch { $_.Exception }
     finally {
       if ($br -ne $null) { $br.Close() }
       if ($fs -ne $null) { $fs.Close() }
     }
   }
   end {
       $xml = [xml](Get-Content $$)
       Out-File $$ -InputObject $xml.document.InnerText -Encoding Default
     }
   }
 }
`

