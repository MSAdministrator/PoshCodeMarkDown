---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6175
Published Date: 2016-01-11t19
Archived Date: 2016-03-18t21
---

# reading dns debug logs - 

## Description

i wrote a small function for parsing a windows dns debug log. you can pipe both log-lines and filenames to this cmdlet. i’ve added 3 different types of log formats for it to handle.

## Comments



## Usage



## TODO



## function

`get-dnsdebuglog`

## Code

`#
 #
 function Get-DNSDebugLog
 {
     <#
     .SYNOPSIS
     This cmdlet parses a Windows DNS Debug log.
 
     .DESCRIPTION
     When a DNS log is converted with this cmdlet it will be turned into objects for further parsing.
 
     .EXAMPLE
     Get-DNSDebugLog -DNSLog ".\Something.log" | Format-Table
 
     Outputs the contents of the dns debug file "Something.log" as a table.
 
     .EXAMPLE
     Get-DNSDebugLog -DNSLog ".\Something.log" | Export-Csv .\ProperlyFormatedLog.csv
 
     Turns the debug file into a csv-file.
 
     .PARAMETER DNSLog
     Path to the DNS log or DNS log data. Allows pipelining from for example Get-ChildItem for files, and supports pipelining DNS log data.
 
     #>
 
     [CmdletBinding()]
     param(
       [Parameter(Mandatory=$false, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
       [Alias('Fullname')]
       [string] $DNSLog = 'StringMode')
 
 
     BEGIN { }
 
     PROCESS {
 
         $TheReverseRegExString='\(\d\)in-addr\(\d\)arpa\(\d\)'
 
         ReturnDNSLogLines -DNSLog $DNSLog | % {
                 if ( $_ -match '^\d\d|^\d/\d' -AND $_ -notlike '*EVENT*' -AND $_ -notlike '* Note: *') {
                     $Date=$null
                     $Time=$null
                     $DateTime=$null
                     $Protocol=$null
                     $Client=$null
                     $SendReceive=$null
                     $QueryType=$null
                     $RecordType=$null
                     $Query=$null
                     $Result=$null
 
                     $Date=($_ -split ' ')[0]
 
                     if ($_ -match ':\d\d AM|:\d\d  PM') {
                         $Time=($_ -split ' ')[1,2] -join ' '
                         $Protocol=($_ -split ' ')[7]
                         $Client=($_ -split ' ')[9]
                         $SendReceive=($_ -split ' ')[8]
                         $RecordType=(($_ -split ']')[1] -split ' ')[1]
                         $Query=($_.ToString().Substring(110)) -replace '\s' -replace '\(\d?\d\)','.' -replace '^\.' -replace "\.$"
                         $Result=(((($_ -split '\[')[1]).ToString().Substring(9)) -split ']')[0] -replace ' '
                     }
                     elseif ($_ -match '^\d\d\d\d\d\d\d\d \d\d:') {
                         $Date=$Date.Substring(0,4) + '-' + $Date.Substring(4,2) + '-' + $Date.Substring(6,2)
                         $Time=($_ -split ' ')[1] -join ' '
                         $Protocol=($_ -split ' ')[6]
                         $Client=($_ -split ' ')[8]
                         $SendReceive=($_ -split ' ')[7]
                         $RecordType=(($_ -split ']')[1] -split ' ')[1]
                         $Query=($_.ToString().Substring(110)) -replace '\s' -replace '\(\d?\d\)','.' -replace '^\.' -replace "\.$"
                         $Result=(((($_ -split '\[')[1]).ToString().Substring(9)) -split ']')[0] -replace ' '
                     }
                     else {
                         $Time=($_ -split ' ')[1]
                         $Protocol=($_ -split ' ')[6]
                         $Client=($_ -split ' ')[8]
                         $SendReceive=($_ -split ' ')[7]
                         $RecordType=(($_ -split ']')[1] -split ' ')[1]
                         $Query=($_.ToString().Substring(110)) -replace '\s' -replace '\(\d?\d\)','.' -replace '^\.' -replace "\.$"
                         $Result=(((($_ -split '\[')[1]).ToString().Substring(9)) -split ']')[0] -replace ' '
                     }
 
                     $DateTime=Get-Date("$Date $Time") -Format 'yyyy-MM-dd HH:mm:ss'
 
 
                     if ($_ -match $TheReverseRegExString) {
                         $QueryType='Reverse'
                     }
                     else {
                         $QueryType='Forward'
                     }
 
                     $returnObj = New-Object System.Object
                     $returnObj | Add-Member -Type NoteProperty -Name Date -Value $DateTime
                     $returnObj | Add-Member -Type NoteProperty -Name QueryType -Value $QueryType
                     $returnObj | Add-Member -Type NoteProperty -Name Client -Value $Client
                     $returnObj | Add-Member -Type NoteProperty -Name SendReceive -Value $SendReceive
                     $returnObj | Add-Member -Type NoteProperty -Name Protocol -Value $Protocol
                     $returnObj | Add-Member -Type NoteProperty -Name RecordType -Value $RecordType
                     $returnObj | Add-Member -Type NoteProperty -Name Query -Value $Query
                     $returnObj | Add-Member -Type NoteProperty -Name Results -Value $Result
 
                     if ($returnObj.Query -ne $null) {
                         Write-Output $returnObj
                     }
                 }
             }
 
     }
 
     END { }
 }
 
 
 
 function ReturnDNSLogLines
 {
 param(
 $DNSLog)
 
 $PathCorrect=try { Test-Path $DNSLog -ErrorAction Stop } catch { $false }
 
     if ($DNSLog -match '^\d\d|^\d/\d' -AND $DNSLog -notlike '*EVENT*' -AND $PathCorrect -ne $true) {
         $DNSLog
     }
     elseif ($PathCorrect -eq $true) {
         Get-Content $DNSLog | % { $_ }
     }
 }
`

