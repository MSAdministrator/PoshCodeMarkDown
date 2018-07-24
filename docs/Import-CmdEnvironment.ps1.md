---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3547
Published Date: 2013-07-26t20
Archived Date: 2016-06-10t15
---

# import-cmdenvironment - 

## Description

invoke the specified command (with parameters) in cmd.exe, and import any environment variable changes back to powershell

## Comments



## Usage



## TODO



## script

`import-cmdenvironment`

## Code

`#
 #
 #
 
 [CmdletBinding()]
 param(
    [Parameter(Position=0,Mandatory=$False,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [Alias("PSPath")]
    [string]$Command = "echo"
 ,
    [Parameter(Position=0,Mandatory=$False,ValueFromRemainingArguments=$true,ValueFromPipelineByPropertyName=$true)]
    [string[]]$Parameters
 )
 	if(Test-Path $Command) { $Command = "`"$(Resolve-Path $Command)`"" }
    $setRE = new-Object System.Text.RegularExpressions.Regex '^(?<var>.*?)=(?<val>.*)$', "Compiled,ExplicitCapture,MultiLine"
    $OFS = " "
    [string]$Parameters = $Parameters
    $OFS = "`n"
    Write-Verbose "EXECUTING: cmd.exe /c `"$Command $Parameters > nul && set`""
 	foreach($match in  $setRE.Matches((cmd.exe /c "$Command $Parameters > nul && set")) | Select Groups) {
       Set-Content Env:\$($match.Groups["var"]) $match.Groups["val"] -Verbose
 	}
 #}
`

