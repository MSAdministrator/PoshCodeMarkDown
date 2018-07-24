---
Author: nathan estell
Publisher: 
Copyright: 
Email: 
Version: 5.2.119
Encoding: ascii
License: cc0
PoshCode ID: 6381
Published Date: 2016-06-14t18
Archived Date: 2016-06-17t08
---

# division puzzle solver - 

## Description

solves this puzzle

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 	.NOTES
 	===========================================================================
 	 Created with: 	SAPIEN Technologies, Inc., PowerShell Studio 2016 v5.2.119
 	 Created on:   	6/14/2016 12:20 PM
 	 Created by:   	Nathan Estell
 	 Organization: 	University Of Michigan ITS MiWorkspace
 	 Filename:     	
 	===========================================================================
 	.DESCRIPTION
 		From http://www.futilitycloset.com/2016/06/09/divide-and-conquer-3/
 
 #>
 $integers = 1..9
 [array]$results = $null
 [array]$resultsUniqueDigits=$null
 foreach ($integer in $integers)
 {
 	foreach ($integer2 in $integers)
 	{
 		[int]$2DigNum = $integer.ToString() + $integer2.ToString()
 		if ($2DigNum%2 -eq 0)
 		{
 			foreach ($integer3 in $integers)
 			{
 				[int]$3DigNum = $2DigNum.ToString() + $integer3.ToString()
 				if ($3DigNum%3 -eq 0)
 				{
 					foreach ($integer4 in $integers)
 					{
 						[int]$4DigNum = $3DigNum.ToString() + $integer4.ToString()
 						if ($4DigNum%4 -eq 0)
 						{ 
 							foreach ($integer5 in $integers)
 							{
 								[int]$5DigNum = $4DigNum.ToString() + $integer5.ToString()
 								if ($5DigNum%5 -eq 0)
 								{
 									foreach ($integer6 in $integers)
 									{
 										[int]$6DigNum = $5DigNum.ToString() + $integer6.ToString()
 										if ($6DigNum%6 -eq 0)
 										{
 											foreach ($integer7 in $integers)
 											{
 												[int]$7DigNum = $6DigNum.ToString() + $integer7.ToString()
 												if ($7DigNum%7 -eq 0)
 												{
 													foreach ($integer8 in $integers)
 													{
 														[int]$8DigNum = $7DigNum.ToString() + $integer8.ToString()
 														if ($8DigNum%8 -eq 0)
 														{
 															foreach ($integer9 in $integers)
 															{
 																[int]$9DigNum = $8DigNum.ToString() + $integer9.ToString()
 																if ($9DigNum%9 -eq 0)
 																{
 																	$results+=$9DigNum
 																}
 															}
 														}
 													}
 												}
 											}
 										}
 									}
 								}
 							}
 						}
 					}
 				}
 			}
 		}
 	}
 }
 <#
 $resultsLength=$results.Length
 if ($resultsLength -eq 1)
 {
 Write-Output "If digits can be used multiple times there is $resultsLength match:"
 }
 if ($resultsLength -gt 1)
 {
 Write-Output "If digits can be used multiple times there are $resultsLength matches:"
 }
 Write-Output $results
 #>
 foreach ($result in $results)
 {
 	$hasUniqueDigits=$true
 	[string]$resultStr=$result
 	for ($i = 0; $i -lt $resultStr.Length; $i++)
 	{
 		for ($j = $i+1; $j -lt $resultStr.Length; $j++)
 		{
 			if ($resultStr[$i] -eq $resultStr[$j])
 			{
 				$hasUniqueDigits=$false
 			}
 		}
 	}
 	if ($hasUniqueDigits)
 	{
 		$resultsUniqueDigits+=$result
 	}
 }
 $resultsUniqueDigitsLength = $resultsUniqueDigits.Length
 if ($resultsUniqueDigitsLength -eq 1)
 {
 	Write-Output "If digits cannot be used multiple times there is $resultsUniqueDigitsLength match:"
 }
 if ($resultsUniqueDigitsLength -gt 1)
 {
 	Write-Output "If digits cannot be used multiple times there are $resultsUniqueDigitsLength matches:"
 }
 
 Write-Output $resultsUniqueDigits
`

