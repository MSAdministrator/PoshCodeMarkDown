---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2374
Published Date: 
Archived Date: 2010-11-23t15
---

# set-iisloglocation - 

## Description

this advanced function will allow you to set the iis log location on a server or servers.  you can specify a single site or perform the task on all sites. also supports -whatif in the function.

## Comments



## Usage



## TODO



## function

`set-iisloglocation`

## Code

`#
 #
 Function Set-IISLogLocation {
 .SYNOPSIS  
     This command will allow you to set the IIS log location on a server or multiple servers.
 .DESCRIPTION
     This command will allow you to set the IIS log location on a server or multiple servers.    
 .PARAMETER computer
     Name of computer to set log location on
 .PARAMETER logdir
     Location to set IIS logs to write to 
 .PARAMETER website
     Name of website to change the log location.       
 .NOTES  
     Name: Set-IISLogLocation
     Author: Boe Prox
     DateCreated: 20Aug2010 
          
 .LINK  
     http://boeprox.wordpress.com
 .EXAMPLE  
     Set-IISLogLocation -computer <server> -logdir "D:\logs"
     
 Description
 -----------
 This command will change the IIS log locations for each website on the server.
 .EXAMPLE  
     Set-IISLogLocation -computer <server> -logdir "D:\logs" -website "Default Web Site"
     
 Description
 -----------
 This command will change the IIS log locations for only the Default Web Site on a server.
           
 #> 
 [cmdletbinding(
     SupportsShouldProcess = $True,
 	DefaultParameterSetName = 'default',
 	ConfirmImpact = 'low'
 )]
 param(
     [Parameter(
         Mandatory = $False,
         ParameterSetName = '',
         ValueFromPipeline = $True)]
         [string]$computer,
     [Parameter(
         Mandatory = $False,
         ParameterSetName = '',
         ValueFromPipeline = $False)]
         [string]$logdir,
     [Parameter(
         Mandatory = $False,
         ParameterSetName = 'site',
         ValueFromPipeline = $False)]
         [string]$website                           
 )
 Process {
     ForEach ($c in $Computer) {
 
             If (Test-Connection -comp $c -count 1) {
                 
                 $sites = [adsi]"IIS://$c/W3SVC"
                 $children = $sites.children
                 ForEach ($child in $children) {
                     Switch ($pscmdlet.ParameterSetName) { 
                        "default" {
                                 If ($child.KeyType -eq "IIsWebServer") {
                                 If ($pscmdlet.ShouldProcess($($child.servercomment))) {
                                     $child.Put("LogFileDirectory",$logdir)
                                     $child.SetInfo()
                                     Write-Host -fore Green "$($child.servercomment): Log location set to $logdir"
                                     }                                   
                                 } 
                             }
                         "site" {
                                 If ($child.KeyType -eq "IIsWebServer" -AND $child.servercomment -eq $website) {
                                 If ($pscmdlet.ShouldProcess($($child.servercomment))) {
                                     $child.Put("LogFileDirectory",$logdir)
                                     $child.SetInfo()
                                     Write-Host -fore Green "$($child.servercomment): Log location set to $logdir"
                                     }                                   
                                 }                             
                             }                                                                                                   
                         }
                     }                        
             }                
         }  
     }
 }
`

