---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6782
Published Date: 2017-03-11t11
Archived Date: 2017-03-19t19
---

# get-website - 

## Description

this script will allow you to query a web site and retrieve information about the web site and whether it is available or not.

## Comments



## Usage



## TODO



## function

`get-website`

## Code

`#
 #
 function Get-WebSite {
 .SYNOPSIS  
     Retrieves information about a website.
 .DESCRIPTION
     Retrieves information about a website.
 .PARAMETER Url
     URL of the website to test access to.
 .PARAMETER UseDefaultCredentials
     Use the currently authenticated user's credentials  
 .PARAMETER Proxy
     Used to connect via a proxy
 .PARAMETER TimeOut
     Timeout to connect to site, in milliseconds
 .PARAMETER Credential
     Provide alternate credentials              
 .NOTES  
     Name: Get-WebSite
     Author: Boe Prox
     DateCreated: 08Feb2011        
 .EXAMPLE  
     Get-WebSite -url "http://www.bing.com"
     
 Description
 ------------
 Returns information about Bing.Com to include StatusCode and type of web server being used to host the site.
 
 #> 
 [cmdletbinding(
 	DefaultParameterSetName = 'url',
 	ConfirmImpact = 'low'
 )]
     Param(
         [Parameter(
             Mandatory = $True,
             Position = 0,
             ParameterSetName = '',
             ValueFromPipeline = $True)]
             [string][ValidatePattern("^(http|https)\://*")]$Url,
         [Parameter(
             Position = 1,
             Mandatory = $False,
             ParameterSetName = 'defaultcred')]
             [switch]$UseDefaultCredentials,
         [Parameter(
             Mandatory = $False,
             ParameterSetName = '')]
             [string]$Proxy,
         [Parameter(
             Mandatory = $False,
             ParameterSetName = '')]
             [Int]$Timeout,
         [Parameter(
             Mandatory = $False,
             ParameterSetName = 'altcred')]
             [switch]$Credential            
                         
         )
 Begin {     
     $psBoundParameters.GetEnumerator() | % { 
         Write-Verbose "Parameter: $_" 
         }
    
     Write-Verbose "Creating the web request object"        
     $webRequest = [net.WebRequest]::Create($url)
     
     If ($PSBoundParameters.ContainsKey('Proxy')) {
         Write-Verbose "Creating proxy address and adding into Web Request"
         $webRequest.Proxy = New-Object -TypeName Net.WebProxy($proxy,$True)
         }
         
     If ($PSBoundParameters.ContainsKey('TimeOut')) {
         Write-Verbose "Setting the timeout on web request"
         $webRequest.Timeout = $timeout
         }        
     
     If ($UseDefaultCredentials) {
         Write-Verbose "Using Default Credentials"
         $webrequest.UseDefaultCredentials = $True
         }
     If ($Credential) {
         Write-Verbose "Prompt for alternate credentials"
         $wc.Credential = (Get-Credential).GetNetworkCredential()
         }            
         
     $then = get-date
     }
 Process {    
     Try {
         $response = $webRequest.GetResponse()
         $now = get-date 
         
         Write-Verbose "Generating report from website connection and response"  
         $report = @{
             URL = $url
             StatusCode = $response.Statuscode -as [int]
             StatusDescription = $response.StatusDescription
             ResponseTime = "$(($now - $then).totalseconds)"
             WebServer = $response.Server
             Size = $response.contentlength
             } 
         }
     Catch {
         $now = get-date
         $errorstring = "$($error[0])"
         
         $report = @{
             URL = $url
             StatusCode = ([regex]::Match($errorstring,"\b\d{3}\b")).value
             StatusDescription = (($errorstring.split('\)')[2]).split('.\')[0]).Trim()
             ResponseTime = "$(($now - $then).totalseconds)" 
             WebServer = $response.Server
             Size = $response.contentlength
             }   
         }
     }
 End {        
     New-Object PSObject -property $report  
     }    
 }
`

