---
Author: william stacey
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2618
Published Date: 2011-04-18t23
Archived Date: 2011-04-20t17
---

# get-tail.ps1 - get-tail.ps1

## Description

gets the last n lines of a file. does scan from end-of-file so works on large files. also has a loop flag that prompts for refresh.

## Comments



## Usage



## TODO



## script

`get-tail`

## Code

`#
  
 function Get-Tail([string]$path = $(throw "Path name must be specified."), [int]$count = 10, [bool]$loop = $false)
 {
 	if ( $count -lt 1 ) {$(throw "Count must be greater than 1.")}
 	function get-last
 	{
 		$lineCount = 0
 		$reader = new-object -typename System.IO.StreamReader -argumentlist $path, $true
 		[long]$pos = $reader.BaseStream.Length - 1
  
 		while($pos -gt 0)
 		{
 			$reader.BaseStream.Position = $pos
 			if ($reader.BaseStream.ReadByte() -eq 10)
 			{
 				if($pos -eq $reader.BaseStream.Length - 1)
 				{
 					$count++
 				}
 				$lineCount++
 				if ($lineCount -ge $count) { break }
 			}
 			$pos--
 		} 
 		
 		if ($lineCount -lt $count)
 		{
 			$reader.BaseStream.Position = 0
 		}
 		
 		while($line = $reader.ReadLine())
 		{
 			$lines += ,$line
 		}
 		
 		$reader.Close()
 		$lines
 	}
  
 	while(1)
 	{
 		get-last
 		if ( ! $loop ) { break }
 		$in = read-host -prompt "Hit [Enter] to tail again or Ctrl-C to exit"
 	}
 }
`

