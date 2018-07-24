---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 927
Published Date: 
Archived Date: 2009-03-15t01
---

# scriptsvn.ps1 - 

## Description

script from svn utility

## Comments



## Usage



## TODO



## script

`split-directive`

## Code

`#
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 $exampleInputFile = @"
 {SourceControl "svn://sccserver/trunk/database/maindb/"}
 {OutputFile "Test2.sql"}
 #{hello world}
 
 {Reference "Procedure/ProcedureA.sql" }
 {Reference "Procedure/ProcedureB.sql" "" "print 'dont forget to do job 222'\n"}
 #{Reference "Procedure/ProcedureC.sql" } -- commented out for now
 {Reference "Procedure/ProcedureD.sql" "43"}	-- extract specific revision file
 
 select 'This is a test'
 select 'What a nice day.'
 	#{Me to}
 --select 'Yyyy'
 "@
 
 $commentStart = "#"
 $comment = "--"
 $version = "3"
 
 $matchDirective=[Regex]"^(\s*)({([^{]+)})(.*)$"
 
 $matchSingleQuoteString=[Regex]@"
 ^([^']*)(['].*?['])([^']*)$
 "@
 
 function split-directive([string]$str)
 {
 	$myMatches = $matchDirective.match($str)
 	if ($myMatches.Success)
 	{
 		return $myMatches.Groups[1], $myMatches.Groups[3], $myMatches.Groups[4]
 	}
 }
 
 function get-commentindex([string]$str)
 {
 	$startCommentIndex = -1
 	$myMatches = $matchSingleQuoteString.match($str)
 
 	{
 		$commentIndex = ([string]$myMatches.Groups[1]).IndexOf($commentStart)
 		if ($commentIndex -gt -1)
 			$startCommentIndex = $myMatches.Groups[1].Index + $commentIndex
 		}
 		else
 		{
 			$commentIndex = ([string]$myMatches.Groups[3]).IndexOf($commentStart)
 			if ($commentIndex -gt -1)
 			{
 				$startCommentIndex = $myMatches.Groups[3].Index + $commentIndex
 			}
 		}
 	}
 	{
 		$startCommentIndex = $str.IndexOf($commentStart)
 	}
 	return $startCommentIndex
 }
 
 function split-comment([string]$str)
 {
 	$startCommentIndex = get-commentindex $str
 	if ($startCommentIndex -gt -1)
 	{
 		return $str.Substring(0, $startCommentIndex), $str.Substring($startCommentIndex)
 	}
 	else
 	{
 		return $str
 	}
 }
 
 ################################################################################
 ################################################################################
 	##
 	################################################################################
 Function Convert-Delimiter([regex]$from,[string]$to) 
 {
    begin
    {
 	  $z = [char](222)
    }
    process
    {
 	  $_ = $_ -replace "^\s+", "" -replace "\s+$", ""
 	  $_ = $_ -replace "(?:`"((?:(?:[^`"]|`"`"))+)(?:`"$from|`"`$))|(?:$from)|(?:((?:.(?!$from))*.)(?:$from|`$))","$z`$1`$2$z$to"
 	  $_ = $_ -replace "$z(?:$to|$z)?`$","$z"
 	  $_ = $_ -replace "`"`"","`"" -replace "`"","`"`"" 
 	  $_ = $_ -replace "$z((?:[^$z`"](?!$to))+)$z($to|`$)","`$1`$2"
 	  $_ = $_ -replace "$z","`"" -replace "$z","`""
 	  $_
    }
 }
 
 Function get-reference([string]$scc, [string]$filePath, [string]$rev, [string]$post, [string]$whiteSpacePre)
 {
 	$svn = 'svn.exe'
 
 	if ($rev -eq $null -or $rev -eq "")
 	{
 		$revResult = @($result -like "Revision: *")
 		if ($revResult.Length -eq 1)
 		{
 			$tmp = $revResult[0]
 			$regexRevision = [regex]"^Revision: (\d+)$"
 			$myMatches = $regexRevision.match($tmp)
 			if ($myMatches.Success)
 			{
 				$rev = $myMatches.Groups[1]
 			}
 		}
 		else
 		{
 			write-error "Cannot find revision for $scc$filePath."
 			exit
 		}
 	}
 	
 	write-host "Reference [$rev] $filePath"
 	$result = & $svn -r $rev cat $scc$filePath
 	 
 	for ($i = 0; $i -lt $result.Length; $i++)
 	{
 		$result[$i] =  $whiteSpacePre + $result[$i]
 	}
 	
 	$fileContent = [string]::join("`r`n", $result)
 	
 	append-outputString "$comment ** Start Reference [$scc] [$filePath] [$rev] [$post]`r`n"; 
 	if ($post -ne $null -and $post -ne "")
 	{
 		$convertedPost = $post -replace "\\n", "`r`n"  -replace "\\t", "`t"
 		append-outputString $convertedPost
 	}
 	append-outputString "$comment ** End Reference [$scc] [$filePath] [$rev] [$post]`r`n"; 
 }
 
 function init-outputString()
 {
 	$global:outputString = ""
 	$global:sourceControl = ""
 	$global:outputFile = ""
 }
 function append-outputString([string]$str)
 {
 	$global:outputString += $str
 }
 function write-outputString([string]$file)
 {
 	write-output $global:outputString | out-file -filePath $file -encoding oem
 }
 
 function execute-directive($splitDirective, [string]$whiteSpacePre)
 {
 	$keyword = $splitDirective[0]
 	switch ($keyword)
 	{
 		"SourceControl"
 		{
 			write-host  "$comment $keyword `"$($splitDirective[1])`""
 			append-outputString "$whiteSpacePre$comment $keyword `"$($splitDirective[1])`""
 			$global:sourceControl = $splitDirective[1]
 		}
 		"OutputFile"	
 		{
 			append-outputString "$whiteSpacePre$comment $keyword `"$($splitDirective[1])`""
 			$global:outputFile = $splitDirective[1]
 		}
 		"Reference"
 		{
 			get-reference $global:sourceControl $splitDirective[1] $splitDirective[2] $splitDirective[3] $whiteSpacePre
 		}
 		default
 		{
 			write-error "Unknown directive `"$keyword`" exiting..."
 			exit
 		}
 	}
 }
 
 function process-directive([string]$directive, [string]$whiteSpacePre)
 {
 	$reDelimit = $directive | Convert-Delimiter " " "!"
 	$splitDirective = $reDelimit.Split("!")
 	
 	for ($i = 0; $i -lt $splitDirective.Length; $i++)
 	{
 		$tmp = $splitDirective[$i]
 		{
 		}
 	}
 	execute-directive $splitDirective $whiteSpacePre
 }
 
 function process-file([string]$inputFile, [string]$outputFile)
 {
 	init-outputString
 	$global:outputFile = $outputFile
 	
 	if (!(test-path -pathType Leaf $inputFile))
 	{
 		write-error "Cannot open input file `"$inputFile`""
 		exit
 	}
 	
 	$outmsg = "$comment ScriptSVN $version processing file `"$inputFile`""
 	write-host $outmsg
 	append-outputString "$outmsg`r`n"
 	
 	foreach ($line in $inputLines)
 	{
 		$activeStr, $commentStr = split-comment $line
 		$whiteSpacePre, $directive, $poststr = split-directive $activeStr
 		if ($directive -eq $null -or $directive.Length -eq 0)
 		{
 		}
 		else
 		{
 			process-directive $directive $whiteSpacePre
 		}
 	
 		if ($commentStr.Length -gt 0)
 		{
 			$afterComment = $commentStr.Substring($commentStart.Length)
 			append-outputString "$comment$afterComment`r`n"
 		}
 		else
 		{
 		}
 	}
 	write-host  "$comment OutputFile `"$($global:outputFile)`""
 	write-outputString $global:outputFile 
 }
 
 $files = @(get-childitem "*.dro")
 
 if ($files.Length -eq 0)
 {
 	write-host "No files found for processing."
 	write-host "This utility processes files ending in `".dro`" "
 }
 
 foreach ($f in $files)
 {
 	$outFile = $f -replace ".dro",".out"
 	process-file $f $outFile
 }
`

