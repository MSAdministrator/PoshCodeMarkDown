---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: utf-8
License: cc0
PoshCode ID: 195
Published Date: 
Archived Date: 2009-03-27t18
---

# import-delimited 2 - 

## Description

convert and import from delimited text files.

## Comments



## Usage



## TODO



## function

`convert-delimiter`

## Code

`#
 #
 ################################################################################
 ################################################################################
 ##
 ################################################################################
 ################################################################################
 Function Convert-Delimiter([regex]$from,[string]$to) 
 { 
    process
    {  
       $_ = $_ -replace "(?:`"((?:(?:[^`"]|`"`"))+)(?:`"$from|`"`$))|(?:((?:.(?!$from))*.)(?:$from|`$))","Ã�`$1`$2Ã�$to" 
       $_ = $_ -replace "Ã�(?:$to|Ã�)?`$","Ã�"
       $_ = $_ -replace "`"`"","`"" -replace "`"","`"`"" 
       $_ = $_ -replace "Ã�((?:[^Ã�`"](?!$to))+)Ã�($to|`$)","`$1`$2"
       write-output ($_ -replace "Ã�","`"")
    }
 }
 
 ################################################################################
 ################################################################################
 ##
 ################################################################################
 
 Function Import-Delimited([regex]$delimiter=",", [string]$PsPath="")
 {
     BEGIN {
         if ($PsPath.Length -gt 0) { 
             write-output ($PsPath | &($MyInvocation.InvocationName) $delimiter); 
         } else {
             $script:tmp = [IO.Path]::GetTempFileName()
             write-debug "Using tempfile $($script:tmp)"
         }
     }
     PROCESS {
         if($_ -and $_.Length -gt 0 ) {
             if(Test-Path $_) {
                 if($delimiter -eq ",") {
                 } else {
                     Get-Content $_ | Convert-Delimiter $delimiter "," | Where-Object { if( $_.StartsWith("--") ) { write-debug "SKIPPING: $_"; $false;} else { $true }} | Add-Content $script:tmp
                 }
             }
             else {
                 if($delimiter -eq ",") {
                 } else {
                     $_ | Convert-Delimiter $delimiter "," | Add-Content $script:tmp
                 }
             }
         }
     }
     END {
         if ($PsPath.Length -eq 0) {
             Write-Output (Import-Delimited $delimiter $script:tmp)
         }
     }
 }
`

