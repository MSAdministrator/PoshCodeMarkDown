---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3184
Published Date: 2012-01-24t20
Archived Date: 2017-03-31t03
---

# get-head - 

## Description

read the first few characters of a file … fast.

## Comments



## Usage



## TODO



## function

`get-head`

## Code

`#
 #
 function Get-Head {
 #.Synopsis
 param(
 [Parameter(Mandatory=$true, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
 [Alias("Path","PSPath")]
 [String]$FilePath, 
 [Parameter(Mandatory=$false)]
 [Int]$count=512, 
 [Parameter(Mandatory=$false)]
 [ValidateSet("UTF8","ASCII")]
 $encoding = "UTF8"
 )
 begin {
    if($encoding -eq "UTF8") {
       $decoder = New-Object System.Text.UTF8Encoding $true
    } else {
       $decoder = New-Object System.Text.AsciiEncoding
    }
    $buffer = New-Object Byte[] $count
 }
 process {
    $Stream = [System.IO.File]::Open($FilePath, 'Open', 'Read')
    $Null = $Stream.Read($buffer,0,$count)
    $decoder.GetString($buffer)
    $Stream.Close()
 }
 }
`

