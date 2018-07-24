---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.7
Encoding: ascii
License: cc0
PoshCode ID: 2931
Published Date: 2011-08-27t19
Archived Date: 2011-11-05t18
---

# paraimpu - 

## Description

the first couple of functions for sending data to paraimpu from powershell.  it’s the first step in letting powershell participate in a network of things…

## Comments



## Usage



## TODO



## module

`send-paraimpu`

## Code

`#
 #
 #
 
 ipmo json, httprest
 
 function Send-Paraimpu {
 #.Synopsis 
 #.Description
 #.Example
 #.Notes 
 param(
    [Hashtable]$Data, 
    [Guid]$Token = "a9988bbd-cb35-4b2d-ba23-69198d36977b"
 )
    $json = New-JSON @{
       token = $Token
       'content-type' = "application/json"
       data = $Data
    }
    http post http://paraimpu.crs4.it/data/new -content $json
 }
 
 function Send-ParaChumby { 
 #.Synopsis 
 #.Description
 #.Example
 param(
    [Parameter(Position=0,ValueFromPipeline=$true,Mandatory=$true)]
    $InputObject,
    [Parameter(Position=1)]
    $Image, 
    [Parameter(Position=2)]
    $Sound = "http://www.frogstar.com/audio/download/14250/gong.mp3",
    [Switch]$Collect,
    [Int]$Width = 30,
    [Guid]$Token = "a9988bbd-cb35-4b2d-ba23-69198d36977b"
 )
 begin {
    if($Collect) {
       $output = New-Object System.Collections.ArrayList
    }
 }
 process {
    if(!$Collect) {
       Send-Paraimpu @{ text = ($InputObject | Out-String -Width $Width | Tee -var dbug) -replace "`r`n","`n"; image = $Image; sound = $Sound } -Token:$Token
       Write-Verbose $dbug
    } else {
       $null = $Output.Add($InputObject)
    }
 }
 end {
    if($Collect) {
       Send-Paraimpu @{ text = ($Output | Out-String -Width $Width | Tee -var dbug) -replace "`r`n","`n"; image = $Image; sound = $Sound } -Token:$Token
       Write-Verbose $dbug
    }
 }}
`

