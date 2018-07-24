---
Author: steven murawski http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 734
Published Date: 2009-12-15t14
Archived Date: 2017-03-12t01
---

# show-adobject - 

## Description

this is a modification of the get-admapobject (http

## Comments



## Usage



## TODO



## script

`write-help`

## Code

`#
 #
 #
 
 param ($ADClass, $MapLayout= "circular", [switch]$help, [switch]$ShowADClass, [switch]$Colorize, [switch]$NoTest)
 
 function Write-Help()
 {
 	$ExampleUsage = @'
 To use this script, you will need the Show-Netmap script from Doug Finke,
 along with the NodeXL DLLs.
 Downloadable from http://www.codeplex.com/NodeXL/Release/ProjectReleases.aspx?ReleaseId=20494
 Usage:
 .\Show-ADObject.ps1 -ADClass [(AD Type Name || Array of AD Type Names}] [-Colorize]
 .\Show-ADObject.ps1 -ShowADClasses
 
 
 
 Example:
 .\Show-ADObject -ADClass group, organizationalunit -Colorize 
 
 '@
 	Write-Host $ExampleUsage
 }
 
 function Test-Prerequisites()
 {
 	$required = @{ShowNetMap = 'Show-NodeXLMap.ps1';
 		NetMapApp = 'Microsoft.NodeXL.ApplicationUtil.dll';
 		NetMapControl = 'Microsoft.NodeXL.Control.dll';
 		NetMapCore =  'Microsoft.NodeXL.Core.dll';
 		NetMapUtil =  'Microsoft.NodeXL.Util.dll';
 		NetMapVisual =  'Microsoft.NodeXL.Visualization.dll'
 		}
 		
 	$report = @()
 	
 	foreach ($key in $required.Keys)
 	{
 		$file = $required[$($key)]
 		if (Test-Path $file )
 		{
 			Write-Debug "Found $file"
 		}
 		else
 		{
 			$report +=  "Missing $file"
 		}
 		
 	}
 	
 	if ($report.count -eq 0)
 	{
 		Write-Debug "All prerequisites were found."
 		return $true
 	}
 	else
 	{
 		Write-Help
 		Write-Host "Missing files: "
 		$report | ForEach-Object { Write-Host "`tMissing $_" }
 		throw "Please move the needed files into the current directory!"
 	}
 
 
 }
 
 if ($NoTest -or (Test-Prerequisites))
 {
 	[Reflection.Assembly]::LoadWithPartialName("System.Drawing")   | Out-Null
 
 	function Get-ParentFromDN()
 	{
 		PROCESS
 		{
 			$root = '^DC=(\w+),DC=(\w+)$'
 			$pattern = '^(OU|CN)=(\w+?),.*?DC=\w+?,DC=\w+?$'
 			$dn = $_
 			
 			
 			if ($dn -notmatch $root)
 			{
 				$dn -replace $pattern, '$2'
 			}
 			else
 			{
 				$dn -replace $root, '$1.$2'
 			}
 		}
 	}
 	
 	function Get-ADObjectClassName()
 	{
 		Get-QADObject -SizeLimit 0 | Select-Object -property @{name='Name';Expression={$_.type}} -unique | Sort-Object name
 	}
 	
 	function Get-ColorList()
 	{
 		[System.Drawing.Color] | Get-Member -memberType property -static | foreach {$_.name}
 	}
 	
 	function Get-RandomColor()
 	{
 		param($ListOfColors)
 		$count = $ListOfColors.count-1
 		$rand = New-Object System.Random
 		$value = $rand.next(0,$count)
 		$ListOfColors[$value]
 	}
 	
 	function Get-ADMapObject()
 	{
 		param($TypesToMap=$(Throw 'One (or more object types as an array) are required to run this function'), [switch]$Color)
 		
 		$script:ColorKey = @{}
 		
 		if ($TypesToMap -is [string])
 		{
 			if ($color)
 			{
 				$MapColor = Get-RandomColor Get-ColorList
 				$script:ColorKey["$TypesToMap"] = $MapColor
 				Get-QADObject -Type $TypesToMap | select Name, @{name='Parent';Expression={$_.ParentContainerDN | Get-ParentFromDN}}, @{name='Color';Expression={$MapColor}}
 			}
 			else
 			{
 				Get-QADObject -Type $TypesToMap | select Name, @{name='Parent';Expression={$_.ParentContainerDN | Get-ParentFromDN}}
 			}
 		}
 		else
 		{
 			$colorList = Get-ColorList
 			foreach ( $type in $TypesToMap )
 			{
 				if ($color)
 				{
 					$MapColor = Get-RandomColor $colorList
 					$colorList = $colorList | ? {$_ -ne $MapColor}
 					$script:ColorKey["$type"] = $MapColor
 					Get-QADObject -Type $type | select Name, @{name='Parent';Expression={$_.ParentContainerDN | Get-ParentFromDN}}, @{name='Color';Expression={$MapColor}}
 				}
 				else
 				{
 					Get-QADObject -Type $type | select Name, @{name='Parent';Expression={$_.ParentContainerDN | Get-ParentFromDN}}
 				}
 			}
 		}
 	}
 	
 	function New-SourceTarget ($s,$t,$c) 
 	{
 		New-Object PSObject |
 			Add-Member -pass noteproperty source $s |
 			Add-Member -pass noteproperty target $t | 
 			Add-Member -pass noteproperty color $c 
 	}
 	
 	if ($help)
 	{
 		Write-Help
 	}
 	else
 	{
 		if ($ShowADClass)
 		{
 			Get-ADObjectClassName
 		}
 		else
 		{
 			. .\Show-NodeXLMap.ps1
 			
 			if ($Colorize)
 			{
 				$MapObjects = Get-ADMapObject -TypesToMap $ADClass -Color |  % { New-SourceTarget $_.Name $_.Parent $_.Color }  
 				Write-Host "Color Key: "
 				$ColorKey
 			}
 			else
 			{
 				$MapObjects = Get-ADMapObject -TypesToMap $ADClass |  % { New-SourceTarget $_.Name $_.Parent }
 			}
 			
 			$MapObjects | Show-NodeXLMap -layoutType $MapLayout
 			
 		}
 	}
 	
 	
 }
`

