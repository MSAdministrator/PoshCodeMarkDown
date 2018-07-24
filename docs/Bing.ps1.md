---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 4375
Published Date: 2013-08-07t21
Archived Date: 2013-08-13t09
---

# bing - 

## Description

a new bing module that supports command-line search against the new datamarket apis.

## Comments



## Usage



## TODO



## module

`search-bing`

## Code

`#
 #
 
 
 param(
   $ApiKey,
 
   [System.Management.Automation.PSCredential]
   [System.Management.Automation.Credential()]
   $Credential = $(if($ApiKey){ Get-Credential -User BingApiKey -Password $ApiKey -Store } else { Get-Credential -User BingApiKey })
 )
 
 Add-Type -Assembly System.Web
 
 $Ofs = " "
 
 $Selectors = @{
   Web = @{
     Format = "{0} {1}"
     Fields = "Title","Url"
   }
   News = @{
     Format = "{0} (from {2}) {1}"
     Fields = "Title","Url","Source"
   }
   Image = @{
     Format = "{0} ({2}x{3}) {1}"
     Fields = "Title","MediaUrl","Width","Height"
   }
   SpellingSuggestions = @{ 
     Format = "{0}"
     Fields = "Value"
   }
 }
 
 function Search-Bing {
   #.Synopsis
   #.Notes
   param(
     [Parameter(Position=0, Mandatory=$true, ValueFromRemainingArguments=$true, ValueFromPipeline=$true)]
     [String[]]$Query,
 
     [ValidateSet("Web","News","Image","SpellingSuggestions")]
     $Noun = "Web",
 
     [ValidateRange(0,5)]
     [int]$Count = 3,
 
     [int]$Skip = 0
   )
   $Query = [Web.HttpUtility]::UrlEncode("'$Query'")
 
   $Search = "https://api.datamarket.azure.com/Bing/Search/${Noun}?Query=${Query}&`$top=$Count&`$skip=$Skip"
 
   $global:BingResults = Invoke-WebRequest -Credential $Credential -Uri $Search
   $global:BingEntries = ([xml]$BingResults.Content).feed.entry.content.properties
 
   $Selector = $Selectors.$Noun.Fields -join "' or local-name() = '"
   $Selector =  "*[local-name() = '$Selector']/text()"
   foreach($entry in $BingEntries) {
     $Selectors.$Noun.Format -f $entry.SelectNodes($Selector).Value
   }
 
 }
`

