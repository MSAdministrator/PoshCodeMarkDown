---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.4
Encoding: utf-8
License: cc0
PoshCode ID: 6063
Published Date: 2016-10-22t14
Archived Date: 2016-08-17t01
---

# import-delimited - 

## Description

convert and import from delimited text files. includes two functions

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
 Function Convert-Delimiter([regex]$from,[string]$to,[switch]$quoteEmpty) 
 {
    begin
    {
       $z = [char](222)
    }
    process
    {
       $_ = $_.Trim()
       $_ = $_ -replace "(^|$from)`"`"($from|`$)","`$1`$2"
       
       $_ = $_ -replace "(?:`"((?:(?:[^`"]|`"`"))+)(?:`"$from|`"`$))|(?:$from)|(?:((?:.(?!$from))*.)(?:$from|`$))","$z`$1`$2$z$to"
       $_ = $_ -replace "$z(?:$to|$z)?`$","$z"
       $_ = $_ -replace "`"`"","`"" -replace "`"","`"`"" 
       $_ = $_ -replace "$z((?:[^$z`"](?!$to))+)$z($to|`$)","`$1`$2"
       if(!$quoteEmpty) {
          $_ = $_ -replace "$to$z$z$to","$to$to"
       }
       $_ = $_ -replace "$z","`"" -replace "$z","`""
       
       $_
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
             Import-Csv $script:tmp
         }
     }
 }
`

