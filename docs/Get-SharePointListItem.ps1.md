---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3710
Published Date: 2012-10-24t07
Archived Date: 2012-12-24t00
---

# get-sharepointlistitem - 

## Description

get items from sharepoint lists

## Comments



## Usage



## TODO



## function

`get-splistitem`

## Code

`#
 #
 #.Example
 #.Example
 #.Example
 function Get-SPListItem {
 [CmdletBinding(DefaultParameterSetName="FromListName")]
 param(
 	[Parameter(Mandatory=$True, Position=0, ParameterSetName="FromListName")]
 	[string]$SharepointUrl,
 	[Parameter(Mandatory=$True, Position=1, ParameterSetName="FromListName")]
 	[String]$Name,
 	[Parameter(Mandatory=$True,ParameterSetName="FromListUrl")]
 	[string]$ListUrl,
 	[String[]]$Property,
 	[Int]$MaxCount = 1000
 )
 	Add-Type -Assembly System.Web.Services
 
 	if($PSCmdlet.ParameterSetName -eq "FromListUrl") {
 		if(!$ListUrl.Contains("/Lists/")) {
 			throw "Can't deduce list name from ListUrl"
 		}
 		$i = $ListUrl.IndexOf("/Lists/")
 		$SharepointUrl = $ListUrl.SubString(0,$i)
 		$ListName = $ListUrl.SubString($i+7, $ListUrl.IndexOf("/",$i+7) - $i - 7)
 		if($ListName -match "%") {
 			Add-Type -Assembly System.Web
 			$ListName = [System.Web.HttpUtility]::UrlDecode($ListName)
 		}
 	}
 
 	$ServiceUrl = $SharepointUrl.Trim("/") + "/_vti_bin/Lists.asmx"
 	Write-Verbose "Sharepoint Service Url: $ServiceUrl"
 	Write-Verbose "Sharepoint List Name: $ListName"
 	
 	$Service = New-WebServiceProxy $ServiceUrl -UseDefaultCred
 	$Service.Url = $ServiceUrl
 
 	trap [System.Web.Services.Protocols.SoapException] {
 		throw $_
 	}
 	
 	$list = $Service.GetList($ListName) 	
 	Write-Verbose "Default List View URL: $(([Uri]$SharepointUrl).GetLeftPart('authority'))$($List.DefaultViewUrl)"
 	$Fields = $list.Fields.Field
 	
 	if($DebugPreference -gt "SilentlyContinue") {
 		$Global:SPListFields = $Fields
 		Write-Debug "Global variable SPListFields set to list fields for debugging purposes"
 	}
 
 	if($Property.Count -gt 0) { 
 		$Fields = $Fields | Where-Object { $Property -contains $_.DisplayName }
 	} else {
 		$Fields = $Fields | Where-Object { $_.FromBaseType -ne "TRUE" -and $_.Hidden -ne "TRUE" }
 	}
 
 	$ViewFields = @()
 	$ObjectProperties = @()
 	foreach($f in $Fields.GetEnumerator()) {
 		$ViewFields += $f.StaticName 
 		$ObjectProperties += @{ Name = $f.DisplayName; Expression = [ScriptBlock]::Create("`$_.`"ows_$($f.StaticName)`"") }
 	}
 
 	[xml]$vf = "<ViewFields>$($ViewFields | ForEach-Object { "<FieldRef Name='$_'/>" })</ViewFields>"
 	Write-Verbose $vf.OuterXml
 	
 	$ListItems = $service.GetListItems($listName, "", [Xml]"<Query/>", $vf, $MaxCount, [Xml]"<QueryOptions/>", "").Data.Row
 
 	if($DebugPreference -gt "SilentlyContinue") {
 		$Global:SPObjectProperties = $ObjectProperties
 		Write-Debug "Global variable SPObjectProperties set to ObjectProperties for debugging purposes"
 	}
 	
 	if($DebugPreference -gt "SilentlyContinue") {
 		$Global:SPListItems = $ListItems
 		Write-Debug "Global variable SPListItems set to list items for debugging purposes"
 	}
 	$ListItems | Select-Object $ObjectProperties
 }
`

