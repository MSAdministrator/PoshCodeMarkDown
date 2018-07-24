---
Author: obscuresec
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3863
Published Date: 2013-01-05t10
Archived Date: 2013-01-09t07
---

# invoke-rickroll - 

## Description

script to open ie to a video of rick astley (or any specified url)

## Comments



## Usage



## TODO



## function

`invoke-rickroll`

## Code

`#
 #
 function Invoke-RickRoll {
 <#
 .SYNOPSIS
 
     Invoke-RickRoll
 
     A RickRoll PowerShell Script :)
 
     Authors: Chris Campbell (@obscuresec)
     License: BSD 3-Clause
 
 .DESCRIPTION
 
     A script to call IE and send it to a URL.
 
 .PARAMETER VideoURL
 
     Specifies a URL to send IE to.
 
 .EXAMPLE
 
     PS C:\>Invoke-RickRoll -VideoURL http://www.obscuresec.com
 #>
 
     [CmdletBinding()] Param(
     [string] $VideoURL = 'http://www.youtube.com/watch?v=dQw4w9WgXcQ'
     )
     
     [string] $CmdString = "$env:SystemDrive\PROGRA~1\INTERN~1\iexplore.exe $VideoURL"
     Write-Verbose "I am now opening IE to $VideoUrl"
     Invoke-Expression $CmdString
 }
`

