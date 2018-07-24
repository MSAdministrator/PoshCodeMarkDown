---
Author: bartekb
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2626
Published Date: 2011-04-22t00
Archived Date: 2011-04-25t16
---

# watch-sg2011leaderboard - 

## Description

my variation for sg 2011 leader board watcher.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 <#
     .Synopsis
       
         Script that will produce table of leaders in a give category.
 
     .Description
 
         Just some fun with regex/ xml. :)
         It will grab SG 2011 Leader Board for a given category.
         Than parse the string, convert it into XML, create custom objects and in the end - produce nice table.
         Not really best practices, but well... it's all about display this time.
         Goes in loop, so it should keep leaderboard.
         And if you miss you name in topX - add it using -Name parameter. :)
 
     .Example
 
         Watch-SG2011LeaderBoard
         Starts using defaults: Advanced category, Top 10 users, 5 seconds delay, and 2011 SG url.
         
     .Example
 
         Watch-SG2011LeaderBoard -Categeroy Beg -Name ScriptingWife
         Show Beginner category, and include ScriptingWife regardless her current position. :)
 
     #>
 
 [CmdletBinding()]
 param (
     [ValidateSet('Adv','Advanced','Beg','Beginner')]
     [string]$Category = 'Adv',
     
     [string]$Name,
     
     [int]$Top = 10,
     
     [int]$Delay = 5,
     
     [string]$URL = 'https://2011sg.poshcode.org/Reports/TopUsers'
 )
 
 $WebClient = New-Object Net.WebClient
 switch -regex ($Category) {
     "^A.*" {
         $URL = $URL + '?filter=Advanced'
     }
     "^B.*" {
         $URL = $URL + '?filter=Beginner'
     }
     default {
         Write-Verbose "Why would I handle alien's/ divine intervention? Isn't parameter validation enough? ;)"
     }
 }
 
 $Title = "$Category Scripting Games Leaderboard"
 
 if ($Name) {
     $Filter = { ($TopDisplayed++ -lt $Top) -or ($_.Name -match $Name) }
 } else {
     $Filter = { $TopDisplayed++ -lt $Top }
 }
 while ($True) {
     
     $WebClient.DownloadString($URL) | Select-String -Pattern '(<table>[\s\S]*?</table>)' | Select -ExpandProperty Matches |
         ForEach-Object {
             
             $Table = $_.Groups[0].Value -replace '&\w+?;' -replace '</?a.*?>' -replace '\s*(?!\w)' -replace '(?<!\w)\s*'
         }
     try {
         $XML = [xml]$Table
     } catch {
         Write-Host "Sorry, could not change to XML. Take a look at error below for more details..."
         Write-Error $_
         exit
     }
         
     
     $TopDisplayed = 0
     
     Clear-Host
     Write-Host $Title
     "=" * $Title.Length | Out-Default
     "Last update: $(Get-Date -DisplayHint Time)"
     $XML.table.tr | Select-Object -Skip 1 | ForEach-Object { 
         New-Object PSObject -Property @{ 
             Name = $_.td[0]
             Total = [double]($_.td[1])
             Rated = [int]($_.td[2])
             Notes = [int]($_.td[3])
         }
     } | Sort-Object -Property Total -Descending | Where-Object $Filter |
         Format-Table Name, Total, Rated, Notes -AutoSize
     Start-Sleep -Seconds $Delay
 }
`

