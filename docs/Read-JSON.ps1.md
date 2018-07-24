---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4866
Published Date: 2014-02-03t07
Archived Date: 2014-02-09t11
---

# read-json - 

## Description

from greg�s repository on github.

## Comments



## Usage



## TODO



## function

`read-json`

## Code

`#
 #
 function Read-JSON {
   <#
     .EXAMPLE
         PS C:\>Read-JSON data.json
     .NOTES
         Author: greg zakharov
   #>
   param(
     [Parameter(Mandatory=$true)]
     [ValidateScript({Test-Path $_})]
     [String]$FileName
   )
   
   begin {
     Add-Type -AssemblyName System.Web.Extensions
     $jss = New-Object Web.Script.Serialization.JavaScriptSerializer
   }
   process {
     try {
       $jss.DeserializeObject((cat (cvpa $FileName)))
     }
     catch [Management.Automation.MethodInvocationException] {
       Write-Host Invalid JSON primitive. -fo Red
     }
   }
   end {}
 }
`

