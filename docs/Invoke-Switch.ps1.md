---
Author: kyle a eppler
Publisher: 
Copyright: 
Email: 
Version: 0.9
Encoding: ascii
License: cc0
PoshCode ID: 3873
Published Date: 2013-01-10t16
Archived Date: 2013-01-22t14
---

# invoke-switch - 

## Description

a function to invoke a switch statement, in order to allow splatting to be used with switch

## Comments



## Usage



## TODO



## function

`invoke-switch`

## Code

`#
 #
 New-Variable castDictionaryEntries
 [System.Func[System.Collections.IEnumerable, System.Collections.Generic.IEnumerable[System.Collections.DictionaryEntry]]] `
     $castDictionaryEntries = [System.Delegate]::CreateDelegate(
         [System.Func[System.Collections.IEnumerable, System.Collections.Generic.IEnumerable[System.Collections.DictionaryEntry]]],
         [System.Linq.Enumerable].GetMethod(
             'Cast', @('Public, Static')).MakeGenericMethod(
                 @([System.Collections.DictionaryEntry])))
 Set-Variable castDictionaryEntries -Option ReadOnly
 
 function Invoke-Switch {
     [CmdletBinding()]
     Param(
         [Parameter(Position=0, Mandatory=$true)]
         [ValidateNotNullOrEmpty()]
         [System.Collections.IDictionary] $CaseMap,
 
         [Parameter(Position=1, Mandatory=$false)]
         [ValidateSet('Regex', 'Wildcard', 'Exact')]
         [string] $SwitchMode,
 
         [Parameter(Position=2, Mandatory=$false)]
         $DefaultCase,
 
         [Parameter(Position=3, Mandatory=$true, ValueFromPipeline=$true)]
         [AllowNull()]
         $InputObject,
 
         [Parameter()]
         [switch] $CaseSensitive)
 
     begin {
         New-Variable mode; [string] $mode = $null
         if($PSBoundParameters.ContainsKey('SwitchMode')) {
             $mode = " -$SwitchMode"
         }
 
         New-Variable caseOption; [string] $caseOption = $null
         if($CaseSensitive.IsPresent) {
             $caseOption = ' -CaseSensitive'
         }
 
         Set-Variable mode, caseOption -Option ReadOnly
 
         New-Variable value
         New-Variable cases; [string] $cases =
             $castDictionaryEntries.Invoke($CaseMap) `
                 | % { $value = $_.Value; return $_.Key } `
                 | repr `
                 | % { return $_ } `
                     -End {
                             if($PSBoundParameters.TryGetValue(
                                 'DefaultCase', [ref] $value))
                             {
                                 Write-Output default
                             }
                         } `
                 | % { return "$_ { Write-Output (,$(repr $value)) }" } `
                 | Join-String -Separator ' '
         Set-Variable cases -Option ReadOnly
 
         New-Variable switchScript; [scriptblock] `
             $switchScript = $PSCmdlet.InvokeCommand.NewScriptBlock(
                 "switch$mode$caseOption (`$InputObject) { $cases }")
         Set-Variable switchScript -Option ReadOnly
 
         $PSCmdlet.WriteDebug($switchScript)
     }
 
     process {
         . $switchScript | % { $PSCmdlet.WriteObject($_) }
     }
 }
`

