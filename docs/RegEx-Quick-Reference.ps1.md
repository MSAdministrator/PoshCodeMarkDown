---
Author: robbie foust (rfoust@duke.edu)
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 759
Published Date: 2009-12-28t20
Archived Date: 2016-04-06t07
---

# regex quick reference - 

## Description

get-regex.ps1 is a regular expression quick reference for .net/c#/powershell.  it provides a quick dump of info in a pscustomobject for quick access from a prompt. it is more complete than what is available in get-help.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 #
 #
 #
 
 function global:get-regex ([switch]$CharRep, [switch]$CharClass, [switch]$Anchors, [switch]$Comments, [switch]$Grouping, [switch]$Replacement)
 	{
 	$CharRepDesc = "Character representations"
 	$CharClassDesc = "Character classes and class-like constructs"
 	$AnchorsDesc = "Anchors and other zero-width tests"
 	$CommentsDesc = "Comments and mode modifiers"
 	$GroupingDesc = "Grouping, capturing, conditional, and control"
 	$ReplacementDesc = "Replacement sequences"
 
 	if (!$CharRep -and !$CharClass -and !$Anchors -and !$Comments -and !$Grouping -and !$Replacement)
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
 
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\a" -pass |
 			add-member noteproperty "Meaning" "Alert (bell), x07." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\b" -pass |
 			add-member noteproperty "Meaning" "Backspace, x08, supported only in character class." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\e" -pass |
 			add-member noteproperty "Meaning" "ESC character, x1B." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\n" -pass |
 			add-member noteproperty "Meaning" "Newline, x0A." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\r" -pass |
 			add-member noteproperty "Meaning" "Carriage return, x0D." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\f" -pass |
 			add-member noteproperty "Meaning" "Form feed, x0C." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\t" -pass |
 			add-member noteproperty "Meaning" "Horizontal tab, x09." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\v" -pass |
 			add-member noteproperty "Meaning" "Vertical tab, x0B." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\0octal" -pass |
 			add-member noteproperty "Meaning" "Character specified by a two-digit octal code." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\xhex" -pass |
 			add-member noteproperty "Meaning" "Character specified by a two-digit hexadecimal code." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\uhex" -pass |
 			add-member noteproperty "Meaning" "Character specified by a four-digit hexadecimal code." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 		$CharRepObj += new-object psobject | add-member noteproperty "Sequence" "\cchar" -pass |
 			add-member noteproperty "Meaning" "Named control character." -pass |
 			add-member noteproperty "Table" $CharRepDesc -pass
 
 		$CharRepObj
 		}
 
 	if ($CharClass -or $all)
 		{
 		$CharClassObj = @()
 
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "[...]" -pass |
 			add-member noteproperty "Meaning" "A single character listed or contained within a listed range." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "[^...]" -pass |
 			add-member noteproperty "Meaning" "A single character not listed and not contained within a listed range." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "." -pass |
 			add-member noteproperty "Meaning" "Any character, except a line terminator (unless single-line mode, s)." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "\w" -pass |
 			add-member noteproperty "Meaning" "Word character." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "\W" -pass |
 			add-member noteproperty "Meaning" "Non-word character." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "\d" -pass |
 			add-member noteproperty "Meaning" "Digit." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "\D" -pass |
 			add-member noteproperty "Meaning" "Non-digit." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "\s" -pass |
 			add-member noteproperty "Meaning" "Whitespace character." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "\S" -pass |
 			add-member noteproperty "Meaning" "Non-whitespace character." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "\p{prop}" -pass |
 			add-member noteproperty "Meaning" "Character contained by given Unicode block or property." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 		$CharClassObj += new-object psobject | add-member noteproperty "Sequence" "\P{prop}" -pass |
 			add-member noteproperty "Meaning" "Character not contained by given Unicode block or property." -pass |
 			add-member noteproperty "Table" $CharClassDesc -pass
 
 		$CharClassObj
 		}
 
 	if ($Anchors -or $all)
 		{
 		$AnchorsObj = @()
 
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "^" -pass |
 			add-member noteproperty "Meaning" "Start of string, or after any newline if in MULTILINE mode." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "\A" -pass |
 			add-member noteproperty "Meaning" "Beginning of string, in all match modes." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "$" -pass |
 			add-member noteproperty "Meaning" "End of string, or before any newline if in MULTILINE mode." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "\Z" -pass |
 			add-member noteproperty "Meaning" "End of string but before any final line terminator, in all match modes." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "\z" -pass |
 			add-member noteproperty "Meaning" "End of string, in all match modes." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "\b" -pass |
 			add-member noteproperty "Meaning" "Boundary between a \w character and a \W character." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "\B" -pass |
 			add-member noteproperty "Meaning" "Not-word-boundary." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "\G" -pass |
 			add-member noteproperty "Meaning" "End of the previous match." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "(?=...)" -pass |
 			add-member noteproperty "Meaning" "Positive lookahead." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "(?!...)" -pass |
 			add-member noteproperty "Meaning" "Negative lookahead." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "(?<=...)" -pass |
 			add-member noteproperty "Meaning" "Positive lookbehind." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 		$AnchorsObj += new-object psobject | add-member noteproperty "Sequence" "(?<!...)" -pass |
 			add-member noteproperty "Meaning" "Negative lookbehind." -pass |
 			add-member noteproperty "Table" $AnchorsDesc -pass
 
 		$AnchorsObj
 		}
 
 	if ($Comments -or $all)
 		{
 		$CommentsObj = @()
 
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "Singleline (s)" -pass |
 			add-member noteproperty "Meaning" "Dot (.) matches any character, including a line terminator." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "Multiline (m)" -pass |
 			add-member noteproperty "Meaning" "^ and $ match next to embedded line terminators." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "IgnorePatternWhitespace (x)" -pass |
 			add-member noteproperty "Meaning" "Ignore whitespace and allow embedded comments starting with #." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "IgnoreCase (i)" -pass |
 			add-member noteproperty "Meaning" "Case-insensitive match based on characters in the current culture." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "CultureInvariant (i)" -pass |
 			add-member noteproperty "Meaning" "Culture-insensitive match." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "ExplicitCapture (n)" -pass |
 			add-member noteproperty "Meaning" "Allow named capture groups, but treat parentheses as non-capturing groups." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "Compiled" -pass |
 			add-member noteproperty "Meaning" "Compile regular expression." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "RightToLeft" -pass |
 			add-member noteproperty "Meaning" "Search from right to left, starting to the left of the start position." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "ECMAScript" -pass |
 			add-member noteproperty "Meaning" "Enables ECMAScript compliance when used with IgnoreCase or Multiline." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "(?imnsx-imnsx)" -pass |
 			add-member noteproperty "Meaning" "Turn match flags on or off for rest of pattern." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "(?imnsx-imnsx:...)" -pass |
 			add-member noteproperty "Meaning" "Turn match flags on or off for the rest of the subexpression." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "(?#...)" -pass |
 			add-member noteproperty "Meaning" "Treat substring as a comment." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 		$CommentsObj += new-object psobject | add-member noteproperty "Sequence" "#..." -pass |
 			add-member noteproperty "Meaning" "Treat rest of line as a comment in /x mode." -pass |
 			add-member noteproperty "Table" $CommentsDesc -pass
 
 		$CommentsObj
 		}
 
 	if ($Grouping -or $all)
 		{
 		$GroupingObj = @()
 
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "(...)" -pass |
 			add-member noteproperty "Meaning" "Grouping. Submatches fill \1,\2,... and `$1,`$2,...." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "\n" -pass |
 			add-member noteproperty "Meaning" "In a regular expression, match what was matched by the nth earlier submatch." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "`$n" -pass |
 			add-member noteproperty "Meaning" "In a replacement string, contains the nth earlier submatch." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "(?<name>...)" -pass |
 			add-member noteproperty "Meaning" "Captures matched substring into group, 'name'." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "(?:...)" -pass |
 			add-member noteproperty "Meaning" "Grouping-only parentheses, no capturing." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "(?>...)" -pass |
 			add-member noteproperty "Meaning" "Disallow backtracking for subpattern." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "...|..." -pass |
 			add-member noteproperty "Meaning" "Alternation; match one or the other." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "*" -pass |
 			add-member noteproperty "Meaning" "Match 0 or more times." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "+" -pass |
 			add-member noteproperty "Meaning" "Match 1 or more times." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "?" -pass |
 			add-member noteproperty "Meaning" "Match 1 or 0 times." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "{n}" -pass |
 			add-member noteproperty "Meaning" "Match exactly n times." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "{n,}" -pass |
 			add-member noteproperty "Meaning" "Match at least n times." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "{x,y}" -pass |
 			add-member noteproperty "Meaning" "Match at least x times, but no more than y times." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "*?" -pass |
 			add-member noteproperty "Meaning" "Match 0 or more times, but as few times as possible." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "+?" -pass |
 			add-member noteproperty "Meaning" "Match 1 or more times, but as few times as possible." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "??" -pass |
 			add-member noteproperty "Meaning" "Match 0 or 1 times, but as few times as possible." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "{n,}?" -pass |
 			add-member noteproperty "Meaning" "Match at least n times, but as few times as possible." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 		$GroupingObj += new-object psobject | add-member noteproperty "Sequence" "{x,y}?" -pass |
 			add-member noteproperty "Meaning" "Match at least x times, no more than y times, but as few times as possible." -pass |
 			add-member noteproperty "Table" $GroupingDesc -pass
 
 		$GroupingObj
 		}
 
 
 	if ($Replacement -or $all)
 		{
 		$ReplacementObj = @()
 
 		$ReplacementObj += new-object psobject | add-member noteproperty "Sequence" "`$1, `$2, ..." -pass |
 			add-member noteproperty "Meaning" "Captured submatches." -pass |
 			add-member noteproperty "Table" $ReplacementDesc -pass
 		$ReplacementObj += new-object psobject | add-member noteproperty "Sequence" "`${name}" -pass |
 			add-member noteproperty "Meaning" "Matched text of a named capture group." -pass |
 			add-member noteproperty "Table" $ReplacementDesc -pass
 		$ReplacementObj += new-object psobject | add-member noteproperty "Sequence" "`$'" -pass |
 			add-member noteproperty "Meaning" "Text before match." -pass |
 			add-member noteproperty "Table" $ReplacementDesc -pass
 		$ReplacementObj += new-object psobject | add-member noteproperty "Sequence" "`$&" -pass |
 			add-member noteproperty "Meaning" "Text of match." -pass |
 			add-member noteproperty "Table" $ReplacementDesc -pass
 		$ReplacementObj += new-object psobject | add-member noteproperty "Sequence" "`$'" -pass |
 			add-member noteproperty "Meaning" "Text after match." -pass |
 			add-member noteproperty "Table" $ReplacementDesc -pass
 		$ReplacementObj += new-object psobject | add-member noteproperty "Sequence" "`$+" -pass |
 			add-member noteproperty "Meaning" "Last parenthesized match." -pass |
 			add-member noteproperty "Table" $ReplacementDesc -pass
 		$ReplacementObj += new-object psobject | add-member noteproperty "Sequence" "`$_" -pass |
 			add-member noteproperty "Meaning" "Copy of original input string." -pass |
 			add-member noteproperty "Table" $ReplacementDesc -pass
 
 		$ReplacementObj
 		}
 
 	}
`

