---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2511
Published Date: 2013-02-17t16
Archived Date: 2013-01-11t06
---

# demo-attributes - 

## Description

a demo of using some build in dotnet attributes to store some powershell metadata (hashtables) and extract that.

## Comments



## Usage



## TODO



## function

`demo-attributes`

## Code

`#
 #
 function demo-attributes
 {
 
 [System.Configuration.SettingsDescription({
     @{something = 1;
       this = 'that';
       array = (1,2,3);
       sub = @{ sub1 = 1 ; sub2 =2 }
     }})]
     
 [CmdletBinding(DefaultParameterSetName="noname")]
 param (
  [Parameter(Position=1,mandatory = $true)]
   [string]$something 
  
 )
 1..10
 }
 $settingstext = ((dir function:\demo-attributes).scriptblock.attributes |Where-Object { $_.typeid -like '*settingsdescription*'  }).description
 $settingsscriptblock = $ExecutionContext.InvokeCommand.NewScriptBlock("DATA {" + $settingstext + "}")
 &$settingsscriptblock
`

