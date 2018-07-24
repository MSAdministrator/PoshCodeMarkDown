---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 426
Published Date: 
Archived Date: 2010-06-30t08
---

# find-string - 

## Description

find-string and highligh-matches work together to do formatted output of matches in strings

## Comments



## Usage



## TODO



## script

`find-string`

## Code

`#
 #
 ########################################################################################################################
 ##
 ########################################################################################################################
 function Find-String {
 PARAM( [regex] $pattern = $(Read-Host "Regular expression search pattern")
      , [string] $filter = "*.*"
 	  , [switch] $recurse
 	  , [switch] $caseSensitive
      , [switch] $summary
 )
 
 $v2 = [bool](new-object Microsoft.PowerShell.Commands.MatchInfo | gm "Matches")
 
 if ($pattern.ToString() -eq "") { throw "Nothing to search for! Please provide a search pattern!" }
 
 if( (-not $caseSensitive) -and (-not $pattern.Options -match "IgnoreCase") ) {
 	$pattern = New-Object regex $pattern.ToString(),@($pattern.Options,"IgnoreCase")
 }
 
 if($summary) {
    [string[]] $summaryText = ""
    $summaryText += "Matches  Lines  FileName"
    $summaryText += "-------  -----  --------"
 
    $totalCount = 0; $totalLines = 0;
 }
 Get-ChildItem -recurse:$recurse -filter:$filter |
    foreach { 
       if($summary) {
          $fileCount = 0; $lineCount = 0;
       }
       if($v2) {
          $select = "Select-String -input `$_ -caseSensitive:`$caseSensitive -pattern:`$pattern -AllMatches"
       } else {
          $select = "Select-String -input `$_ -caseSensitive:`$caseSensitive -pattern:`$pattern"
       } 
       
       iex $select | foreach {
          if(!$v2) {
             Add-Member -Input $_ -MemberType NoteProperty -Name "Matches" -Value $pattern.Matches($_.Line) -ErrorAction SilentlyContinue
          }
 
          Write-Output $_
          
          if($summary) {
    		   $lineCount++
    		   $fileCount += $_.Matches.Count
          }
 	   }
       if($summary) {
          $totalCount += $fileCount;
          $totalLines += $lineCount;
          if( $fileCount -gt 0) {
             $summaryText += "{0,7}  {1,5}  {2}" -f $fileCount, $lineCount, $_
          }
       }
    }
 if($summary) {
    $summaryText += "-------  -----  -----"
    $summaryText += "{0,7}  {1,5}  TOTALS" -f $totalCount, $totalLines
    $summaryText += ""
    $summaryText | Out-Host
 }
 
 }
 
 function Highlight-Matches
 {
    PARAM ([Microsoft.PowerShell.Commands.MatchInfo[]]$match, [Switch]$Passthru)
    BEGIN {
       if($match) { 
          $match | Write-Host-HighlightMatches
       }
    }
    PROCESS { 
       if($_ -and ($_ -isnot [Microsoft.PowerShell.Commands.MatchInfo])) {
          throw "Expected Microsoft.PowerShell.Commands.MatchInfo objects on the pipeline"
       } elseif($_) {
          Write-Host ("{0} {1,5} {2,3}: " -f $_.FileName, "($($_.LineNumber))", $_.matches.Count) -nonewline
          $index = 0;
          foreach($match in $_.Matches) {
             Write-Host $_.line.SubString($index, $match.Index - $index) -nonewline
             Write-Host $match.Value -ForegroundColor Red -nonewline
             $index = $match.Index + $match.Length
          }
          if($index -lt $_.line.Length) {
             Write-Host $_.line.SubString($index) -nonewline
          }
          Write-Host
          
          if($Passthru) { $_ }
       }
    }
 }
`

