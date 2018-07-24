---
Author: enter your zip code here
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6608
Published Date: 2017-11-04t08
Archived Date: 2017-04-24t05
---

# get-fileencoding - 

## Description

get-fileencoding function determines encoding by looking at byte order mark (bom).

## Comments



## Usage



## TODO



## script

`get-fileencoding`

## Code

`#
 #
 <#
 .SYNOPSIS
 Gets file encoding.
 .DESCRIPTION
 The Get-FileEncoding function determines encoding by looking at Byte Order Mark (BOM).
 .EXAMPLE
 Get-ChildItem  *.ps1 | select FullName, @{n='Encoding';e={Get-FileEncoding $_.FullName}} | where {$_.Encoding -ne 'ASCII'}
 This command gets ps1 files in current directory where encoding is not ASCII
 .EXAMPLE
 Get-ChildItem  *.ps1 | select FullName, @{n='Encoding';e={Get-FileEncoding $_.FullName}} | where {$_.Encoding -ne 'ASCII'} | foreach {(get-content $_.FullName) | set-content $_.FullName -Encoding ASCII}
 Same as previous example but fixes encoding using set-content
 #>
 function Get-FileEncoding
 {
     [CmdletBinding()] Param (
      [Parameter(Mandatory = $True, ValueFromPipelineByPropertyName = $True)] [string]$Path
     )
 
     [byte[]]$byte = get-content -Encoding byte -ReadCount 4 -TotalCount 4 -Path $Path
 
     if ( $byte[0] -eq 0xef -and $byte[1] -eq 0xbb -and $byte[2] -eq 0xbf )
     { Write-Output 'UTF8' }
     elseif ($byte[0] -eq 0xfe -and $byte[1] -eq 0xff)
     { Write-Output 'Unicode' }
     elseif ($byte[0] -eq 0 -and $byte[1] -eq 0 -and $byte[2] -eq 0xfe -and $byte[3] -eq 0xff)
     { Write-Output 'UTF32' }
     elseif ($byte[0] -eq 0x2b -and $byte[1] -eq 0x2f -and $byte[2] -eq 0x76)
     { Write-Output 'UTF7'}
     else
     { Write-Output 'ASCII' }
 }
`

