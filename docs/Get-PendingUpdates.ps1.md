---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5721
Published Date: 2016-02-02t12
Archived Date: 2016-03-16t21
---

# get-pendingupdates - 

## Description

this function will allow you to query local or remote computer/s and determine if there are pending wsus updates that need to be installed. a report is returned that can be exported to a csv file if desired.

## Comments



## Usage



## TODO



## function

`get-pendingupdates`

## Code

`#
 #
 Function Get-PendingUpdates { 
   .SYNOPSIS   
     Retrieves the updates waiting to be installed from WSUS   
   .DESCRIPTION   
     Retrieves the updates waiting to be installed from WSUS  
   .PARAMETER Computer 
     Computer or computers to find updates for.   
   .EXAMPLE   
    Get-PendingUpdates 
     
    Description 
    ----------- 
    Retrieves the updates that are available to install on the local system 
   .NOTES 
   Author: Boe Prox                                           
   Date Created: 05Mar2011                                           
 #> 
       
 [CmdletBinding( 
     DefaultParameterSetName = 'computer' 
     )] 
 param( 
     [Parameter( 
         Mandatory = $False, 
         ParameterSetName = '', 
         ValueFromPipeline = $True)] 
         [string[]]$Computer               
     )     
 Begin { 
     $scriptdir = { Split-Path $MyInvocation.ScriptName �Parent } 
     Write-Verbose "Location of function is: $(&$scriptdir)" 
     $psBoundParameters.GetEnumerator() | ForEach-Object { Write-Verbose "Parameter: $_" } 
     If (!$PSBoundParameters['computer']) { 
         Write-Verbose "No computer name given, using local computername" 
         [string[]]$computer = $Env:Computername 
         } 
     Write-Verbose "Creating report collection" 
     $report = @()     
     } 
 Process { 
     ForEach ($c in $Computer) { 
         Write-Verbose "Computer: $($c)" 
         If (Test-Connection -ComputerName $c -Count 1 -Quiet) { 
             Try { 
                 Write-Verbose "Creating COM object for WSUS Session" 
                 $updatesession =  [activator]::CreateInstance([type]::GetTypeFromProgID("Microsoft.Update.Session",$c)) 
                 } 
             Catch { 
                 Write-Warning "$($Error[0])" 
                 Break 
                 } 
  
             Write-Verbose "Creating COM object for WSUS update Search" 
             $updatesearcher = $updatesession.CreateUpdateSearcher() 
  
             Write-Verbose "Searching for WSUS updates on client" 
             $searchresult = $updatesearcher.Search("IsInstalled=0")     
              
             Write-Verbose "Verifing that updates are available to install" 
             If ($searchresult.Updates.Count -gt 0) { 
                 Write-Verbose "Found $($searchresult.Updates.Count) update\s!" 
                 $count = $searchresult.Updates.Count 
                  
                 Write-Verbose "Iterating through list of updates" 
                 For ($i=0; $i -lt $Count; $i++) { 
                     $update = $searchresult.Updates.Item($i) 
                      
                     Write-Verbose "Checking to see that update has been downloaded" 
                     If ($update.IsDownLoaded -eq "True") {  
                         Write-Verbose "Auditing updates"   
                         $temp = "" | Select Computer, Title, KB,IsDownloaded 
                         $temp.Computer = $c 
                         $temp.Title = ($update.Title -split('\('))[0] 
                         $temp.KB = (($update.title -split('\('))[1] -split('\)'))[0] 
                         $temp.IsDownloaded = "True" 
                         $report += $temp                
                         } 
                     Else { 
                         Write-Verbose "Update has not been downloaded yet!" 
                         $temp = "" | Select Computer, Title, KB,IsDownloaded 
                         $temp.Computer = $c 
                         $temp.Title = ($update.Title -split('\('))[0] 
                         $temp.KB = (($update.title -split('\('))[1] -split('\)'))[0] 
                         $temp.IsDownloaded = "False" 
                         $report += $temp 
                         } 
                     } 
                  
                 } 
             Else { 
                 Write-Verbose "No updates to install." 
                  
                 $temp = "" | Select Computer, Title, KB,IsDownloaded 
                 $temp.Computer = $c 
                 $temp.Title = "NA" 
                 $temp.KB = "NA" 
                 $temp.IsDownloaded = "NA" 
                 $report += $temp 
                 } 
             } 
         Else { 
             Write-Warning "$($c): Offline" 
              
             $temp = "" | Select Computer, Title, KB,IsDownloaded 
             $temp.Computer = $c 
             $temp.Title = "NA" 
             $temp.KB = "NA" 
             $temp.IsDownloaded = "NA" 
             $report += $temp             
             } 
         }  
     } 
 End { 
     Write-Output $report 
     }     
 }
`

