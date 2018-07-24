---
Author: robert
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6811
Published Date: 2017-03-21t15
Archived Date: 2017-03-25t17
---

# get-fileencoding utf8bom - 

## Description

get-fileencoding function determines encoding by looking at byte order mark (bom).

## Comments



## Usage



## TODO



## function

`get-fileencoding`

## Code

`#
 #
 <#
  $byte[0] -eq 0xef -and $byte[1] -eq 0xbb -and $byte[2] -eq 0xbf Gets files that are UTF-8 with BOM
 #>
 function Get-FileEncoding
 {
     [CmdletBinding()] Param (
      [Parameter(Mandatory = $True, ValueFromPipelineByPropertyName = $True)] [string]$Path
     )
 
     [byte[]]$byte = get-content -Encoding byte -ReadCount 4 -TotalCount 4 -Path $Path
 
     if($byte[0] -eq 0xef -and $byte[1] -eq 0xbb -and $byte[2] -eq 0xbf)
     {Write-Output 'UTF-8-BOM'}
     elseif ($byte[0] -eq 0xfe -and $byte[1] -eq 0xff)
     { Write-Output 'Unicode' }
     elseif ($byte[0] -eq 0 -and $byte[1] -eq 0 -and $byte[2] -eq 0xfe -and $byte[3] -eq 0xff)
     { Write-Output 'UTF32' }
     elseif ($byte[0] -eq 0x2b -and $byte[1] -eq 0x2f -and $byte[2] -eq 0x76)
     { Write-Output 'UTF7'}
     else
     { Write-Output 'ASCII or UTF-8' }
 }
`

