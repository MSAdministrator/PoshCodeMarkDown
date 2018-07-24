---
Author: smobutter
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2526
Published Date: 2011-02-25t23
Archived Date: 2016-04-20t01
---

# note, open notepad++ - 

## Description

just a slight remake of a previous ‘edit-file’ advanced function.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function Note
 {
 	<#
 	.Synopsis
 		Opens Notepad++
 	.Description
 		Opens Notepad++
 	.Parameter File
 		File name(s) to open, accepts wildcards. (absolute or relative path name)
 	.Parameter MultiInstance
 		Launch another Notepad++ instance
 	.Parameter PluginsOff
 		Launch Notepad++ without plugins, good for debugging cause
 		of a crash, Notepad++ or plugins?
 	.Parameter Language
 		Launch Notepad++ by applying indicated language to the file to open.
 	.Parameter SessionOff
 		Launch Notepad++ without any session. (without loading 
 		files opened when notepad++ was last in use.)
 	.Parameter TabBarOff
 		Launch Notepad++ without tabbar.
 	.Example
 		Note -Language xml -MultiInstance -TabBarOff -File C:\Script\MyScript.ps1
 			Opens file 'MyScript.ps1' as xml file in a new instance
 			of Notepad++ with no tab bar.
 	.Link
 		http://notepad-plus.sourceforge.net/uk/cmdLine-HOWTO.php
 	.Link
 		http://notepad-plus.sourceforge.net/
 	.Link
 		http://poshcode.org/notepad++lexer/
 	.Link
 		http://sourceforge.net/apps/mediawiki/notepad-plus/index.php?title=Command_Line_Switches
 	.Notes
 		Most of this script is courtesy of Joel Bennet @ PoshCode. Added a couple features to handle the lack
 		of a -File parameter as well as creation of a file.
 	#>
 
 [CmdletBinding()]
 	Param
 	(
 		[Parameter(ValueFromPipeline=$true,Position=1)]
 		[Alias("FileName","LitteralPath","Path")]
 		[string[]]
 		$File
 	,
 		[Parameter()]
 		[string]$Language
 	,
 		[Parameter()]
 		[Switch]$MultiInstance
 	,
 		[Parameter()]
 		[Switch]$PluginsOff
 	,
 		[Parameter()]
 		[Switch]$SessionOff
 	,
 		[Parameter()]
 		[Switch]$TabBarOff
 	)
 
 	BEGIN
 	{
 		$NPP = "C:\Program Files (x86)\Notepad++\Notepad++.exe"
 		$Param = @(
 			if($Language)		{"-l$Language"}
 			if($MultiInstance)	{"-multiInst"}
 			if($PluginsOff)		{"-noPlugins"}
 			if($SessionOff)		{"-nosession"}
 			if($TabBarOff)		{"-notabbar"}
 			" "
 		)-join " "
 	}
 	
 	Process
 	{
 		###
 		###
 		if($File -eq $null){
 			Write-Host "hmmm....`nOpening Notepad++" -foregroundcolor "green"
 			Write-Verbose "$NPP $param"
 			[void][Diagnostics.Process]::Start($NPP,$param).WaitForInputIdle(500)
 		}elseif(Test-Path $File){
 			ForEach($Path in $File){
 			ForEach($f in Convert-Path (Resolve-Path $Path)){
 				$parameters = $param + """" + $f + """"
 				Write-Verbose "$NPP $parameters"
 				[void][Diagnostics.Process]::Start($NPP,$parameters).WaitForInputIdle(500)
 			}
 			}
 		###
 		###
 		}elseif(!(Test-Path $File)){
 			$Title = "File did not exist."
 			$Message = "Would you like to attempt to create?"
 			$Yes = New-Object System.Management.Automation.Host.ChoiceDescription "&Yes",`
 				"Passes file name to Notepad++ to attempt creation."
 			$No = New-Object System.Management.Automation.Host.ChoiceDescription "&No",`
 				"Does not pass file name to Notepad++. Face it, you misspelled something =("
 			$Options = [System.Management.Automation.Host.ChoiceDescription[]]($Yes, $No)
 			$Result = $Host.UI.PromptForChoice($Title, $Message, $Options, 0)
 			Switch($Result){
 				0{
 					$Parameters = $Param + $File
 					Write-Verbose "$NPP $Parameters"
 					[void][Diagnostics.Process]::Start($NPP,$Parameters).WaitForInputIdle(500)
 				}
 				1{
 					Write-Host "Yes, well..." -foregroundcolor "Green"
 				}
 			}
 		}
 	}
 }
`

