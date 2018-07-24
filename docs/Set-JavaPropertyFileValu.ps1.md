---
Author: michael craig
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4629
Published Date: 2015-11-21t18
Archived Date: 2015-05-04t22
---

# set-javapropertyfilevalu - 

## Description

function to update a property value within a java properties file

## Comments



## Usage



## TODO



## function

`set-javapropertyfilevalue`

## Code

`#
 #
 function Set-JavaPropertyFileValue($propFile, $propName, $propValue){
 
 	[string]$finalFile=""
 	
 	if (test-path $propFile){
 		write-verbose "Found property file $propFile"
 		write-verbose "Read propfile"
 		$file = gc $propfile
 		
 		foreach($line in $file){
 			if ((!($line.StartsWith('#'))) -and
 				(!($line.StartsWith(';'))) -and
 				(!($line.StartsWith(";"))) -and
 				(!($line.StartsWith('`'))) -and
 				(($line.Contains('=')))){
 				
 				write-verbose "Searching for: $propName"
 				$property=$line.split('=')[0]
 				write-verbose "Looking: $property" 
 					
 					if ($propName -eq $property){
 						write-verbose "found property name"
 						$line="$propName=$propValue"
 					}
 				}
 			$finalFile+="$line`n"
 		}
 		
 		$finalFile | out-file "$propFile" -encoding "ASCII"
 		
 	}else
 	{
 		write-error "Java Property file: $propFile not found"
 	}
 }
`

