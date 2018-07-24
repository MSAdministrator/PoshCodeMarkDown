---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2414
Published Date: 2011-12-18t15
Archived Date: 2017-04-09t11
---

# receive-stream - 

## Description

a simple stream-reader implementation suitable for simple interactive script task …

## Comments



## Usage



## TODO



## function

`receive-stream`

## Code

`#
 #
 function Receive-Stream {
 #.Synopsis
 #.Description
 #.Param reader
 #.Param fileName
 #.Param encoding
 param( [System.IO.Stream]$reader, $fileName, $encoding = [System.Text.Encoding]::GetEncoding( $null ) )
    
 	if($fileName) {
 		try {
 			$writer = new-object System.IO.FileStream $fileName, ([System.IO.FileMode]::Create)
 		} catch {
 			Write-Error "Failed to create output stream. The error was: '$_'"
 			return $false
 		}
 	} else {
 		[string]$output = ""
 	}
 	   
 	[byte[]]$buffer = new-object byte[] 4096
 	[int]$total = [int]$count = 0
 
 	try {
 		$count = $reader.Read($buffer, 0, $buffer.Length)
 
 		while ($count -gt 0) {
 			if($fileName) {
 				$writer.Write($buffer, 0, $count)
 			} else {
 				$output += $encoding.GetString($buffer, 0, $count)
 			}
 			$count = $reader.Read($buffer, 0, $buffer.Length)
 		} 
 	} catch {
 		Write-Error "Failed to write stream. The error was: '$_'"
 		return $false
 	}
 
 	$reader.Close()
 	$writer.Close()
 	if(!$fileName) 
 		{ return $output }
 	else
 		{ return $true }
 }
`

