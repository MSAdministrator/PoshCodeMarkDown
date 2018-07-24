---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5939
Published Date: 2016-07-16t19
Archived Date: 2016-08-26t03
---

# demo-pipepline - 

## Description

just a simple learning tool …

## Comments



## Usage



## TODO



## function

`demo-pipeline`

## Code

`#
 #
 function Demo-Pipeline {
    #.Synopsis
    #.Example
    [CmdletBinding()]
    param (
       [Parameter(Position=0)]
       [ConsoleColor]$Color,
 
       [Parameter(Position=1,ValueFromPipeline=$true)]
       [String[]]$InputObject,
 
       [Switch]$OutputFromBegin,
 
       [Switch]$OutputFromEnd
    )
    begin {
       Write-Verbose "ENTER Begin $Color"
       if ($PSBoundParameters.ContainsKey("InputObject")) {
          Write-Warning "We don't process InputObject in begin, because that wouldn't be PowerShelly"
       }
 
       Write-Host "BEGIN ($Color)" -Foreground $Color
 
       $HasOutput = $false
       
       if($OutputFromBegin) {
          $HasOutput = $true
          Write-Output "$Color BEGIN OUTPUT"
       }
 
 
       Write-Verbose "EXIT Begin $Color"
    }
    process {
       Write-Verbose "ENTER Process $Color"
       Write-Host "PROCESS: $InputObject ($Color)" -Fore $Color
 
       Write-Output $InputObject
 
       if(!$HasOutput -and !$OutputFromEnd) {
          $HasOutput = $true
          Write-Output "$Color PROCESS OUTPUT"
       }
 
       Write-Verbose "EXIT Process $Color"
    }
    end {
       Write-Verbose "ENTER End $Color"
 
       Write-Host "END: $InputObject ($Color)" -Fore $Color
 
       if($OutputFromEnd) {
          $HasOutput = $true
          Write-Output "$Color END OUTPUT"
       }
 
       Write-Verbose "EXIT End $Color"
    }
 }
`

