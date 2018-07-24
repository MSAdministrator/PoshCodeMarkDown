---
Author: pedro genil
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: utf-8
License: cc0
PoshCode ID: 3786
Published Date: 2012-11-26t00
Archived Date: 2012-11-30t08
---

# usr mailbox in exch 07 - 

## Description

sacamos un listado de los mailbox en una organizacion exchange 2007. y realizamos una compresion del resultado

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 If ((Get-PSSnapin | where {$_.Name -match "Exchange.Management"}) -eq $null)
 {
 	Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Admin
 }
 if (-not (test-path "$env:ProgramFiles\7-Zip\7z.exe")) {throw "$env:ProgramFiles\7-Zip\7z.exe needed"} 
 set-alias sz "$env:ProgramFiles\7-Zip\7z.exe"
 $filePath = 'F:\Scripts\users_Acount\'
 $fecha = get-date 
 $fecha= $fecha.toString("yyyyMMdd")
 $filedate = $fecha
 $info = Get-Mailbox -resultsize unlimited -ignoredefaultscope |select database,displayname,samaccountname,PrimarySmtpAddress,EmailAddresses -expandproperty EmailAddresses | out-file F:\Scripts\users_Acount\$filedate.txt
 $files = Get-ChildItem -Recurse -Path $filePath | Where-Object { $_.name -eq "$fecha.txt" }
 
 sz a "F:\Scripts\users_Acount\$fecha.zip" "$filepath\$files"
 remove-item "F:\Scripts\users_Acount\$filedate.txt"
 #{
 
        
 
 #}
`

