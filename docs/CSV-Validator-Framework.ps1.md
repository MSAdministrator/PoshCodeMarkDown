---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1665
Published Date: 2012-02-25t14
Archived Date: 2012-12-29t04
---

# csv validator framework - 

## Description

a simple csv validator framework supporting external rulesets. run

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param(
     [string]$Path = $(throw "require CSV path!"),
     [string]$RulesetPath = $(throw "require Ruleset path!")
 )
 
 $csvFiles = Resolve-Path -Path $Path
 
 if ((Test-Path (Resolve-Path $RulesetPath))) {
     
     $ruleset = (& $RulesetPath)
     
     write-host "Ruleset has $($ruleset.firstrow.keys.count) first row rule(s)"
     write-host "Ruleset has $($ruleset.allrows.keys.count) all rows rule(s)"
 
     if (-not ($ruleset.firstrow)) {
         write-warning "ruleset $rulesetpath has no rules for first row (list/table validation)"
     }
     
     if (-not ($ruleset.allrows)) {
         throw "ruleset $rulesetpath has no rules for allrows (row validation)"
     }
     
 } else {
     throw "Could not find ruleset $rulesetpath"
 }
 
 write-verbose "Found $($csvfiles.length) file(s)"
 
 $csvFiles | where-object {
         
     [io.path]::GetExtension( $_.providerpath ) -eq ".csv"
     
     } | foreach-object { 
     
         write-host "processing CSV $_"
         
         $firstRow = $true
         $rowcount = 0
         
         import-csv $_.path | foreach-object {
             
             write-host "row $rowcount"
             $currentRow = $_
             
             if ($firstRow) {
                 
                 $ruleset.FirstRow.Keys | % {
                     write-host -nonewline "Processing rule $_ ... "
                     
                     if ((& $ruleset.firstrow[$_] $currentRow)) {
                     
                         write-host -fore green "passed"
                     
                     } else {
                         write-host -fore red "failed"
                     
                     }
                 }
                 
                 $firstRow = $false
             }
             
             $ruleset.AllRows.Keys | % {
                 write-host -nonewline "Processing rule $_ ... "
 
                 if ((& $ruleset.AllRows[$_] $currentRow)) {
                 
                     write-host -fore green "passed"
                 
                 } else {
                     write-host -fore red "failed"                
                 }
             }
             
             $rowcount++
         }
     }
 
 @{
     FirstRow = @{
             ruleVerifyColumns = {
                 param($row)
                 
                 $columns = @("u_logon_name",
                   "u_user_security_password",
                   "g_user_id_changed_by",
                   "i_account_status",
                   "d_date_registered",
                   "d_date_last_changed",
                   "d_date_created")
                 
                 $count = 0
                 
                     if ($row.psobject.members.Match($_)) { $count++ }
                 }            
             
             $count -eq $columns.length
         };
     };
     
     AllRows = @{
         ruleLogonNameLengthGreaterThan8 = {
             param($row)        
             $row.u_logon_name -gt 8
         };
         
         ruleChangedByIsValidGuid = {
             param($row)
 
             try {
                [guid]$row.g_user_id_changed_by > $null
                
                 $true
             
             } catch { $false }
         };
    }
 }
 
`

