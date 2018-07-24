---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2452
Published Date: 2011-01-08t22
Archived Date: 2016-03-19t00
---

# read-hostwithprompt.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Read user input, with choices restricted to the list of options you
 provide.
 
 .EXAMPLE
 
 PS >$caption = "Please specify a task"
 PS >$message = "Specify a task to run"
 PS >$option = "&Clean Temporary Files","&Defragment Hard Drive"
 PS >$helptext = "Clean the temporary files from the computer",
 >>              "Run the defragment task"
 >>
 PS >$default = 1
 PS >Read-HostWithPrompt $caption $message $option $helptext $default
 
 Please specify a task
 Specify a task to run
 [C] Clean Temporary Files  [D] Defragment Hard Drive  [?] Help
 (default is "D"):?
 C - Clean the temporary files from the computer
 D - Run the defragment task
 [C] Clean Temporary Files  [D] Defragment Hard Drive  [?] Help
 (default is "D"):C
 0
 
 #>
 
 param(
     $Caption = $null,
 
     $Message = $null,
 
     [Parameter(Mandatory = $true)]
     $Option,
 
     $HelpText = $null,
 
     $Default = 0
 )
 
 Set-StrictMode -Version Latest
 
 $choices = New-GenericObject `
     Collections.ObjectModel.Collection `
     Management.Automation.Host.ChoiceDescription
 
 for($counter = 0; $counter -lt $option.Length; $counter++)
 {
     $choice = New-Object Management.Automation.Host.ChoiceDescription `
         $option[$counter]
 
     if($helpText -and $helpText[$counter])
     {
         $choice.HelpMessage = $helpText[$counter]
     }
 
     $choices.Add($choice)
 }
 
 $host.UI.PromptForChoice($caption, $message, $choices, $default)
`

