---
Author: daniel
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4186
Published Date: 2013-05-29t09
Archived Date: 2013-06-12t03
---

# sys. adm - 

## Description

get-netview

## Comments



## Usage



## TODO



## function

`get-netview`

## Code

`#
 #
 function Get-NetView  {
 	param($Computer)
 	
 	$Result = @()
 	$ses = NET.EXE VIEW $Computer
 
 	$ColLength = -1
 	$start = -1
 	for($i = 0; $i -lt $ses.Count;$i++)
 	{
 		$temp =$null
 		$temp = $ses[$i]
 		if($temp -like "-----------------*")
 		{
 			$start = $i+1
 		}
 		if($temp -eq "The command completed successfully.")
 		{
 			$start = -1
 		}
 		if($temp -like "Share name*Type*")
 		{
 
 			$end = $temp.IndexOf("Type")
 			$ColLength = $end
 		}
 		
 		if($i -ge $start -and $start -ne -1)
 		{
 			$parts =$temp.Substring(0,$ColLength)
 			$share = ("\\" + $Computer + "\" + $parts.Trim())
 			
 			$obj = New-Object PSObject
 			$obj | Add-Member -MemberType NoteProperty -Name "Share" -Value $share
 			$Result += $obj
 		}
 	}
 	
 	
 	return $Result
 }
 Get-NetView DFSROOT01 | ft -AutoSize
`

