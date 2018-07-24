---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1095
Published Date: 
Archived Date: 2009-10-25t15
---

# out-colormatchinfo - 

## Description

formats and highlights matches in a matchinfo object which

## Comments



## Usage



## TODO



## script

`get-relativepath`

## Code

`#
 #
 <#
 .Synopsis
 	Highlights MatchInfo objects similar to the output from grep.
 .Description
 	Highlights MatchInfo objects similar to the output from grep.
 #>
 param ( 
 	[Parameter(Mandatory=$true, ValueFromPipeline=$true)] 
 	[Microsoft.PowerShell.Commands.MatchInfo] $match
 )
 
 begin {}
 process { 
 	function Get-RelativePath([string] $path) {
 		$path = $path.Replace($pwd.Path, '')
 		if ($path.StartsWith('\') -and (-not $path.StartsWith('\\'))) { 
 			$path = $path.Substring(1) 
 		}
 		$path
 	}
 
 	function Write-PathAndLine($match) {
 		Write-Host (Get-RelativePath $match.Path) -foregroundColor DarkMagenta -nonewline
 		Write-Host ':' -foregroundColor Cyan -nonewline
 		Write-Host $match.LineNumber -foregroundColor DarkYellow
 	}
 
 	function Write-HighlightedMatch($match) {
 		$index = 0
 		foreach ($m in $match.Matches) {
 			Write-Host $match.Line.SubString($index, $m.Index - $index) -nonewline
 			Write-Host $m.Value -ForegroundColor Red -nonewline
 			$index = $m.Index + $m.Length
 		}
 		if ($index -lt $match.Line.Length) {
 			Write-Host $match.Line.SubString($index) -nonewline
 		}
 		''
 	}
 	
 	Write-PathAndLine $match
 
 	$match.Context.DisplayPreContext
 
 	Write-HighlightedMatch $match
 
 	$match.Context.DisplayPostContext
 	''
 }
 end {}
`

