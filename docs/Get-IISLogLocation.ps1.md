---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2373
Published Date: 
Archived Date: 2010-11-23t15
---

# get-iisloglocation - 

## Description

this advanced function can be run against a single server or multiple servers to find the location of the log files for each website configured in iis. this has been tested against ii6 and iis7.

## Comments



## Usage



## TODO



## function

`get-iisloglocation`

## Code

`#
 #
 Function Get-IISLogLocation {
 .SYNOPSIS  
     This function can be ran against a server or multiple servers to locate
     the log file location for each web site configured in IIS.
 .DESCRIPTION
     This function can be ran against a server or multiple servers to locate
     the log file location for each web site configured in IIS.    
 .PARAMETER computer
     Name of computer to query log file location.
 .NOTES  
     Name: Get-IISLogLocation
     Author: Boe Prox
     DateCreated: 11Aug2010 
          
 .LINK  
     http://boeprox.wordpress.com
 .EXAMPLE  
 Get-IISLogLocation -computer 'server1'
 
 Description
 -----------
 This command will list the IIS log location for each website configured on 'Server1'
           
 #> 
 [cmdletbinding(
     SupportsShouldProcess = $True,
 	DefaultParameterSetName = 'computer',
 	ConfirmImpact = 'low'
 )]
 param(
     [Parameter(
         Mandatory = $False,
         ParameterSetName = 'computer',
         ValueFromPipeline = $True)]
         [string[]]$computer      
 )
 Begin {
     $report = @()
     }
 Process {
     ForEach ($c in $Computer) {
 
             If (Test-Connection -comp $c -count 1) {
                 
                 $sites = [adsi]"IIS://$c/W3SVC"
                 $children = $sites.children
                 ForEach ($child in $children) {
                     
                     If ($child.KeyType -eq "IIsWebServer") {
                         $temp = "" | Select Server, WebSite, LogLocation
                         $temp.Server = $c
                         $temp.WebSite = $child.ServerComment
                         $temp.LogLocation = $child.LogFileDirectory                           
                         }                                     
                     $report += $temp                        
                     }
             }                
         } 
     }
 End {
     $report
     }
 }
`

