---
Author: claudio spizzi
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3607
Published Date: 2013-09-03t15
Archived Date: 2016-08-11t17
---

# robocopy wrapper - 

## Description

this function performs an robocopy command with the well known parameters `source`, `target`, `files` and `optinos`. the robocopy result is converted to a easy to read psobject.

## Comments



## Usage



## TODO



## function

`copy-robocopy`

## Code

`#
 #
 
 function Copy-Robocopy
 {
 	param
 	(
 		[Parameter(Position=0, Mandatory=$true)]
 		[String] $Source,
 		
 		[Parameter(Position=1, Mandatory=$true)]
 		[String] $Target,
 		
 		[Parameter(Position=2, Mandatory=$false)]
 		[String[]] $Files = @("*.*"),
 		
 		[Parameter(Position=3, Mandatory=$false)]
 		[String] $Options = ""
 	)
 	
 	$Result = @{
 		"Params" = @{}
 		"Timing" = @{}
 		"Result" = @{}
 	}
 	
 	$LogFile = $ENV:TEMP + "\Robocopy." + (Get-Date -UFormat "%Y%m%d.%H%M%S") + "." + (Get-Random -Minimum 1000 -Maximum 9999) + ".log"
 	
 	$Options = $Options.Replace('/NJH', "")
 	$Options = $Options.Replace('/NJS', "")
 	
 	$Duration = Measure-Command { Start-Process -FilePath 'ROBOCOPY' -ArgumentList ('"' + $Source + '" "' + $Target + '" "' + [String]::Join('" "', $Files) + '" ' + $Options) -RedirectStandardOutput $LogFile -NoNewWindow -Wait }
 	
 	$Robocopy = Get-Content -Path $LogFile
 	
 	$LineHeaderFirst = (0..($Robocopy.Count - 1) | Where-Object { $Robocopy[$_] -eq "-------------------------------------------------------------------------------" })[0]
 	$LineHeaderLast  = (0..($Robocopy.Count - 1) | Where-Object { $Robocopy[$_] -eq "------------------------------------------------------------------------------" })[0]
 	$LineFooterFirst = (($Robocopy.Count - 1)..0 | Where-Object { $Robocopy[$_] -eq "------------------------------------------------------------------------------" })[0]
 	$LineFooterLast  = ($Robocopy.Count - 1)
 	
 	$Result["Params"]["Source"]  = ([String]($Robocopy[$LineHeaderFirst + 5].Split(":", 2)[1].Trim()))
 	$Result["Params"]["Target"]  = ([String]($Robocopy[$LineHeaderFirst + 6].Split(":", 2)[1].Trim()))
 	$Result["Params"]["Files"]   = ([String]::Join(", ", (($LineHeaderFirst + 8)..($LineHeaderLast - 4) | ForEach-Object { $Robocopy[$_].Split(":", 2)[-1].Trim() })))
 	$Result["Params"]["Options"] = ([String]($Robocopy[$LineHeaderLast - 2].Split(":", 2)[1].Trim()))
 	
 	$Result["Timing"]["Duration"] = ([String]($Duration))
 	$Result["Timing"]["Started"]  = ([String](Get-Date -Date $Robocopy[$LineHeaderFirst + 4].Split(":", 2)[1].Trim()))
 	$Result["Timing"]["Ended"]    = ([String](Get-Date -Date $Robocopy[$LineFooterLast - 1].Split(":", 2)[1].Trim()))
 	
 	for ($Line = ($LineFooterFirst + 3); $Line -le ($LineFooterFirst + 6); $Line++)
 	{
 		$Key = $Robocopy[$Line].Substring(02,07).Trim()
 		$Result["Result"][$Key] = @{}
 		
 		for ($Col = 10; $Col -le 60; $Col += 10)
 		{
 			$Title = $Robocopy[$LineFooterFirst + 2].Substring($Col, 10).Trim()
 			$Title = $Title.Substring(0,1).ToUpper() + $Title.Substring(1).ToLower()
 			
 			$Result["Result"][$Key][$Title] = $Robocopy[$Line].Substring($Col, 10).Trim()
 		}
 	}
 	
 	return $Result
 }
`

