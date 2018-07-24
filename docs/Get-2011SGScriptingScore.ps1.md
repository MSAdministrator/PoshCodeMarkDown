---
Author: mike hammond
Publisher: 
Copyright: 
Email: 
Version: 20.88
Encoding: ascii
License: cc0
PoshCode ID: 2622
Published Date: 2011-04-21t14
Archived Date: 2011-04-25t16
---

# get-2011sgscriptingscore - 

## Description

function to retrieve score information for the 2011 microsoft scripting games.

## Comments



## Usage



## TODO



## function

`get-2011sgscriptingscore`

## Code

`#
 #
 Function Get-2011SGScriptingScore ([Parameter(Mandatory=$True)][STRING]$Contestant) {
 
    $WebClient = New-Object System.Net.WebClient
 
    $Results = $WebClient.DownloadString("https://2011sg.poshcode.org/Reports/TopUsers?filter=Beginner")+$WebClient.DownloadString
 
 ("https://2011sg.poshcode.org/Reports/TopUsers?filter=Advanced")
 
    $meIndex = $results.indexof($Contestant)
 
    if($meIndex -eq -1) {
       write-output "Contestant name `"$Contestant`" not found"
    } 
    
    else 
    
    {
 
       $myName = $results.substring($meIndex,$Contestant.Length).trim()
       $remainingtext = $results.substring($meIndex + $Contestant.length)
       $remainingtext = $remainingtext.substring($remainingtext.indexof(">")+1)
       $remainingtext = $remainingtext.substring($remainingtext.indexof(">")+1)
       $remainingtext = $remainingtext.substring($remainingtext.indexof(">")+1)
    
       $myscore = $remainingtext.substring(0,$remainingtext.indexof("<")).trim()
       $remainingtext = $remainingtext.substring($remainingtext.indexof(">")+1)
       $remainingtext = $remainingtext.substring($remainingtext.indexof(">")+1)
    
       $myScripts = $remainingtext.substring(0,$remainingtext.indexof("<")).trim()
       $remainingtext = $remainingtext.substring($remainingtext.indexof(">")+1)
       $remainingtext = $remainingtext.substring($remainingtext.indexof(">")+1)
    
       $myRatings = $remainingtext.substring(0,$remainingtext.indexof("<")).trim()
 
       write-output "Contestant `"$Myname`" has scored $MyScore points from $MyScripts scripts that have accumulated $MyRatings 
 
 ratings"
    }
 
 <#
 .Synopsis
 Retrieves Scripting Games score data for a given contestant
 
 .Description
 Retrieves HTML from the PoshCode.org site from both the beginning and advanced leaderboards and contatenates the full text output.  
 
 The text is then searched for the name of the contestant, and relevant data is retrieved using horrible, horrible text matching code.
 
 .Parameter Contestant
 Contestant is a String parameter used to identify the participant in the scripting games.  This is a case-insensitive match against 
 
 the name under which the user registered for the Scripting Games
 
 .Example
 Get-2011SGScriptingScore "ScriptingWife"
 Contestant "ScriptingWife" has scored 20.88 points from 9 scripts that have accumulated 24 ratings
 
 .Example
 Get-2011SGScriptingScore "NoSuchPlayer"
 Contestant name "NoSuchPlayer" not found
 #>
 
 }
`

