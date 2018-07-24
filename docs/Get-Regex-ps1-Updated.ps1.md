---
Author: robbie foust (rfoust@duke.edu)
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 5395
Published Date: 2016-09-03t14
Archived Date: 2016-05-19t03
---

# get-regex.ps1 updated - 

## Description

modified robbie foust’s get-regex.ps1 to streamline code.  this version requires powershell 3.0.

## Comments



## Usage



## TODO



## function

`get-regex`

## Code

`#
 #
 #
 #
 #
 #
 #
 
 function Get-Regex
 {
     [CmdletBinding()]
     [OutputType([PSCustomObject])]
     Param
     (
         [switch]$CharRep,
         [switch]$CharClass,
         [switch]$Anchors,
         [switch]$Comments,
         [switch]$Grouping,
         [switch]$Replacement
     )
 
     function New-RegexRef ($Sequence,$Meaning,$Table)
     {
         [PSCustomObject]@{
             Sequence = $Sequence
             Meaning  = $Meaning
             Table    = $Table
         }
     }
 
 	$CharRepDesc     = "Character representations"
 	$CharClassDesc   = "Character classes and class-like constructs"
 	$AnchorsDesc     = "Anchors and other zero-width tests"
 	$CommentsDesc    = "Comments and mode modifiers"
 	$GroupingDesc    = "Grouping, capturing, conditional, and control"
 	$ReplacementDesc = "Replacement sequences"
 
 	if (-not $CharRep  -and -not $CharClass  -and
             -not $Anchors  -and -not $Comments   -and
             -not $Grouping -and -not $Replacement)
 	{
 		$all = $true
 	}
 	else
 	{
 		$all = $false
 	}
 
     if ($CharRep -or $all)
     {
         $CharRepObj = @()
 
         $CharRepObj += New-RegexRef -Sequence "\a"      -Table $CharRepDesc -Meaning "Alert `(bell`), x07."
         $CharRepObj += New-RegexRef -Sequence "\b"      -Table $CharRepDesc -Meaning "Backspace, x08, supported only in character class."
         $CharRepObj += New-RegexRef -Sequence "\e"      -Table $CharRepDesc -Meaning "ESC character, x1B."
         $CharRepObj += New-RegexRef -Sequence "\n"      -Table $CharRepDesc -Meaning "Newline, x0A."
         $CharRepObj += New-RegexRef -Sequence "\r"      -Table $CharRepDesc -Meaning "Carriage return, x0D."
         $CharRepObj += New-RegexRef -Sequence "\f"      -Table $CharRepDesc -Meaning "Form feed, x0C."
         $CharRepObj += New-RegexRef -Sequence "\t"      -Table $CharRepDesc -Meaning "Horizontal tab, x09."
         $CharRepObj += New-RegexRef -Sequence "\v"      -Table $CharRepDesc -Meaning "Vertical tab, x0B."
         $CharRepObj += New-RegexRef -Sequence "\0octal" -Table $CharRepDesc -Meaning "Character specified by a two-digit octal code."
         $CharRepObj += New-RegexRef -Sequence "\xhex"   -Table $CharRepDesc -Meaning "Character specified by a two-digit hexadecimal code."
         $CharRepObj += New-RegexRef -Sequence "\uhex"   -Table $CharRepDesc -Meaning "Character specified by a four-digit hexadecimal code."
         $CharRepObj += New-RegexRef -Sequence "\cchar"  -Table $CharRepDesc -Meaning "Named control character."
 
         $CharRepObj
     }
 
     if ($CharClass -or $all)
     {
         $CharClassObj = @()
 
         $CharClassObj += New-RegexRef -Sequence "[...]"    -Table $CharClassDesc -Meaning "A single character listed or contained within a listed range."
         $CharClassObj += New-RegexRef -Sequence "[^...]"   -Table $CharClassDesc -Meaning "A single character not listed and not contained within a listed range."
         $CharClassObj += New-RegexRef -Sequence "."        -Table $CharClassDesc -Meaning "Any character, except a line terminator (unless single-line mode, s)."
         $CharClassObj += New-RegexRef -Sequence "\w"       -Table $CharClassDesc -Meaning "Word character."
         $CharClassObj += New-RegexRef -Sequence "\W"       -Table $CharClassDesc -Meaning "Non-word character."
         $CharClassObj += New-RegexRef -Sequence "\d"       -Table $CharClassDesc -Meaning "Digit."
         $CharClassObj += New-RegexRef -Sequence "\D"       -Table $CharClassDesc -Meaning "Non-digit."
         $CharClassObj += New-RegexRef -Sequence "\s"       -Table $CharClassDesc -Meaning "Whitespace character."
         $CharClassObj += New-RegexRef -Sequence "\S"       -Table $CharClassDesc -Meaning "Non-whitespace character."
         $CharClassObj += New-RegexRef -Sequence "\p{prop}" -Table $CharClassDesc -Meaning "Character contained by given Unicode block or property."
         $CharClassObj += New-RegexRef -Sequence "\P{prop}" -Table $CharClassDesc -Meaning "Character not contained by given Unicode block or property."
     }
 
     if ($Anchors -or $all)
     {
         $AnchorsObj = @()
 
         $AnchorsObj += New-RegexRef -Sequence "^"        -Table $AnchorsDesc -Meaning "Start of string, or after any newline if in MULTILINE mode."
         $AnchorsObj += New-RegexRef -Sequence "\A"       -Table $AnchorsDesc -Meaning "Beginning of string, in all match modes."
         $AnchorsObj += New-RegexRef -Sequence "$"        -Table $AnchorsDesc -Meaning "End of string, or before any newline if in MULTILINE mode."
         $AnchorsObj += New-RegexRef -Sequence "\Z"       -Table $AnchorsDesc -Meaning "End of string but before any final line terminator, in all match modes."
         $AnchorsObj += New-RegexRef -Sequence "\z"       -Table $AnchorsDesc -Meaning "End of string, in all match modes."
         $AnchorsObj += New-RegexRef -Sequence "\b"       -Table $AnchorsDesc -Meaning "Boundary between a \w character and a \W character."
         $AnchorsObj += New-RegexRef -Sequence "\B"       -Table $AnchorsDesc -Meaning "Not-word-boundary."
         $AnchorsObj += New-RegexRef -Sequence "\G"       -Table $AnchorsDesc -Meaning "End of the previous match."
         $AnchorsObj += New-RegexRef -Sequence "(?=...)"  -Table $AnchorsDesc -Meaning "Positive lookahead."
         $AnchorsObj += New-RegexRef -Sequence "(?!...)"  -Table $AnchorsDesc -Meaning "Negative lookahead."
         $AnchorsObj += New-RegexRef -Sequence "(?<=...)" -Table $AnchorsDesc -Meaning "Positive lookbehind."
         $AnchorsObj += New-RegexRef -Sequence "(?<!...)" -Table $AnchorsDesc -Meaning "Negative lookbehind."
 
         $AnchorsObj
     }
 
     if ($Comments -or $all)
     {
         $CommentsObj = @()
 
         $CommentsObj += New-RegexRef -Sequence "Singleline (s)"              -Table $CommentsDesc -Meaning "Dot (.) matches any character, including a line terminator."
         $CommentsObj += New-RegexRef -Sequence "Multiline (m)"               -Table $CommentsDesc -Meaning "^ and $ match next to embedded line terminators."
         $CommentsObj += New-RegexRef -Sequence "IgnorePatternWhitespace (x)" -Table $CommentsDesc -Meaning "Ignore whitespace and allow embedded comments starting with #."
         $CommentsObj += New-RegexRef -Sequence "IgnoreCase (i)"              -Table $CommentsDesc -Meaning "Case-insensitive match based on characters in the current culture."
         $CommentsObj += New-RegexRef -Sequence "CultureInvariant (i)"        -Table $CommentsDesc -Meaning "Culture-insensitive match."
         $CommentsObj += New-RegexRef -Sequence "ExplicitCapture (n)"         -Table $CommentsDesc -Meaning "Allow named capture groups, but treat parentheses as non-capturing groups."
         $CommentsObj += New-RegexRef -Sequence "Compiled"                    -Table $CommentsDesc -Meaning "Compile regular expression."
         $CommentsObj += New-RegexRef -Sequence "RightToLeft"                 -Table $CommentsDesc -Meaning "Search from right to left, starting to the left of the start position."
         $CommentsObj += New-RegexRef -Sequence "ECMAScript"                  -Table $CommentsDesc -Meaning "Enables ECMAScript compliance when used with IgnoreCase or Multiline."
         $CommentsObj += New-RegexRef -Sequence "(?imnsx-imnsx)"              -Table $CommentsDesc -Meaning "Turn match flags on or off for rest of pattern."
         $CommentsObj += New-RegexRef -Sequence "(?imnsx-imnsx:...)"          -Table $CommentsDesc -Meaning "Turn match flags on or off for the rest of the subexpression."
         $CommentsObj += New-RegexRef -Sequence "(?#...)"                     -Table $CommentsDesc -Meaning "Treat substring as a comment."
         $CommentsObj += New-RegexRef -Sequence "#..."                        -Table $CommentsDesc -Meaning "Treat rest of line as a comment in /x mode."
 
         $CommentsObj
     }
 
     if ($Grouping -or $all)
     {
         $GroupingObj = @()
 
         $GroupingObj += New-RegexRef -Sequence "(...)"        -Table $GroupingDesc -Meaning "Grouping. Submatches fill \1,\2,... and `$1,`$2,...."
         $GroupingObj += New-RegexRef -Sequence "\n"           -Table $GroupingDesc -Meaning "In a regular expression, match what was matched by the nth earlier submatch."
         $GroupingObj += New-RegexRef -Sequence "`$n"          -Table $GroupingDesc -Meaning "In a replacement string, contains the nth earlier submatch."
         $GroupingObj += New-RegexRef -Sequence "(?<name>...)" -Table $GroupingDesc -Meaning "Captures matched substring into group, 'name'."
         $GroupingObj += New-RegexRef -Sequence "(?:...)"      -Table $GroupingDesc -Meaning "Grouping-only parentheses, no capturing."
         $GroupingObj += New-RegexRef -Sequence "(?>...)"      -Table $GroupingDesc -Meaning "Disallow backtracking for subpattern."
         $GroupingObj += New-RegexRef -Sequence "...|..."      -Table $GroupingDesc -Meaning "Alternation; match one or the other."
         $GroupingObj += New-RegexRef -Sequence "*"            -Table $GroupingDesc -Meaning "Match 0 or more times."
         $GroupingObj += New-RegexRef -Sequence "+"            -Table $GroupingDesc -Meaning "Match 1 or more times."
         $GroupingObj += New-RegexRef -Sequence "?"            -Table $GroupingDesc -Meaning "Match 1 or 0 times."
         $GroupingObj += New-RegexRef -Sequence "{n}"          -Table $GroupingDesc -Meaning "Match exactly n times."
         $GroupingObj += New-RegexRef -Sequence "{n,}"         -Table $GroupingDesc -Meaning "Match at least n times."
         $GroupingObj += New-RegexRef -Sequence "{x,y}"        -Table $GroupingDesc -Meaning "Match at least x times, but no more than y times."
         $GroupingObj += New-RegexRef -Sequence "*?"           -Table $GroupingDesc -Meaning "Match 0 or more times, but as few times as possible."
         $GroupingObj += New-RegexRef -Sequence "+?"           -Table $GroupingDesc -Meaning "Match 1 or more times, but as few times as possible."
         $GroupingObj += New-RegexRef -Sequence "??"           -Table $GroupingDesc -Meaning "Match 0 or 1 times, but as few times as possible."
         $GroupingObj += New-RegexRef -Sequence "{n,}?"        -Table $GroupingDesc -Meaning "Match at least n times, but as few times as possible."
         $GroupingObj += New-RegexRef -Sequence "{x,y}?"       -Table $GroupingDesc -Meaning "Match at least x times, no more than y times, but as few times as possible."
 
         $GroupingObj
     }
 
     if ($Replacement -or $all)
     {
         $ReplacementObj = @()
 
         $ReplacementObj += New-RegexRef -Sequence "`$1, `$2, ..." -Table $ReplacementDesc -Meaning "Captured submatches."
         $ReplacementObj += New-RegexRef -Sequence "`$'"           -Table $ReplacementDesc -Meaning "Text before match."
         $ReplacementObj += New-RegexRef -Sequence "`$&"           -Table $ReplacementDesc -Meaning "Text of match."
         $ReplacementObj += New-RegexRef -Sequence "`$'"           -Table $ReplacementDesc -Meaning "Text after match."
         $ReplacementObj += New-RegexRef -Sequence "`$+"           -Table $ReplacementDesc -Meaning "Last parenthesized match."
         $ReplacementObj += New-RegexRef -Sequence "`$_"           -Table $ReplacementDesc -Meaning "Last parenthesized match."
 
         $ReplacementObj
     }
 }
 
 write-Host 'Use the command "Get-Regex" to get a Regular Expression Quick Reference for .NET/C#/Powershell.'
`

