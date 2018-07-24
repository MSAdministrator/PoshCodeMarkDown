---
Author: andy schneider
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2660
Published Date: 2012-05-06t11
Archived Date: 2017-02-12t19
---

# new-choice - 

## Description

new-choice update

## Comments



## Usage



## TODO



## function

`new-choice`

## Code

`#
 #
 
 function New-Choice {
 <#
 	.SYNOPSIS
 		The New-Choice function is used to provide extended control to a script author who writing code 
         that will prompt a user for information.
 
 	.PARAMETER  Choices
 		An Array of Choices, ie Yes, No and Maybe
 
 	.PARAMETER  Caption
 		Caption to present to end user
 
 	.EXAMPLE
 		PS C:\> New-Choice -Choices 'Yes','No' -Caption "PowerShell makes choices easy"
 		
 	.NOTES
 		Author: Andy Schneider
 		Date: 5/6/2011
 #>
 
 [CmdletBinding()]
 param(
 		
 	[Parameter(Position=0, Mandatory=$True, ValueFromPipeline=$True)]
 	$Choices,
 		
 	[Parameter(Position=1)]
 	$Caption,
     
 	[Parameter(Position=2)]
 	$Message    
 	
 )
 	
 process {
         
         $resulthash += @{}
         for ($i = 0; $i -lt $choices.count; $i++) 
             {
         	   
                $ChoiceDescriptions += @(New-Object System.Management.Automation.Host.ChoiceDescription ("&" + $choices[$i]))
                $resulthash.$i = $choices[$i]
             }
         $AllChoices = [System.Management.Automation.Host.ChoiceDescription[]]($ChoiceDescriptions)
         $result = $Host.UI.PromptForChoice($Caption,$Message, $AllChoices, 1)
         $resulthash.$result -replace "&", ""
         }         
 }
 
 new-choice -Choices "yes","no","maybe"
`

