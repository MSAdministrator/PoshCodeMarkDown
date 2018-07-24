---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4935
Published Date: 2014-02-27t14
Archived Date: 2017-03-01t10
---

# get-mimetype - 

## Description

another way to retrieve mime type

## Comments



## Usage



## TODO



## function

`get-mimetype`

## Code

`#
 #
 function Get-MimeType {
   <#
     .NOTES
         Author: greg zakharov
   #>
   param(
     [Parameter(Mandatory=$true, ValueFromPipeLine=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$FileName
   )
   
   begin {
     Add-Type -AssemblyName System.Web
     $FileName = cvpa $FileName
   }
   process {
     ([AppDomain]::CurrentDomain.GetAssemblies() | ? {
       $_.FullName.Contains('System.Web')
     }).GetType(
       'System.Web.MimeMapping'
     ).GetMethod(
       'GetMimeMapping', [Reflection.BindingFlags]40
     ).Invoke(
       $null, @($FileName)
     )
   }
   end {}
 }
`

