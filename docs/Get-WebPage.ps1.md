---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2498
Published Date: 2011-02-11t20
Archived Date: 2016-03-23t02
---

# get-webpage - 

## Description

this function allows you to download the contents of a webpage to display on a powershell console. &#65279;also included is a switch to show the size of the web page.

## Comments



## Usage



## TODO



## function

`get-webpage`

## Code

`#
 #
 function Get-WebPage {
 .SYNOPSIS  
    Downloads web page from site.
 .DESCRIPTION
    Downloads web page from site and displays source code or displays total bytes of webpage downloaded
 .PARAMETER Url
     URL of the website to test access to.
 .PARAMETER UseDefaultCredentials
     Use the currently authenticated user's credentials  
 .PARAMETER Proxy
     Used to connect via a proxy
 .PARAMETER Credential
     Provide alternate credentials 
 .PARAMETER ShowSize
     Displays the size of the downloaded page in Kilobytes                 
 .NOTES  
     Name: Get-WebPage
     Author: Boe Prox
     DateCreated: 08Feb2011        
 .EXAMPLE  
     Get-WebPage -url "http://www.bing.com"
     
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
             ParameterSetName = 'altcred')]
             [switch]$Credential,
         [Parameter(
             Mandatory = $False,
             ParameterSetName = '')]
             [switch]$ShowSize                        
                         
         )
 Begin {     
     $psBoundParameters.GetEnumerator() | % { 
         Write-Verbose "Parameter: $_" 
         }
    
     Write-Verbose "Creating web client object"
     $wc = New-Object Net.WebClient 
     
     If ($PSBoundParameters.ContainsKey('Proxy')) {
         Write-Verbose "Creating proxy address and adding into Web Request"
         $wc.Proxy = New-Object -TypeName Net.WebProxy($proxy,$True)
         }       
     
     If ($PSBoundParameters.ContainsKey('UseDefaultCredentials')) {
         Write-Verbose "Using Default Credentials"
         $webrequest.UseDefaultCredentials = $True
         }
     If ($PSBoundParameters.ContainsKey('Credentials')) {
         Write-Verbose "Prompt for alternate credentials"
         $wc.Credential = (Get-Credential).GetNetworkCredential()
         }         
         
     }
 Process {    
     Try {
         If ($ShowSize) {
             Write-Verbose "Downloading web page and determining size"
             "{0:N0}" -f ($wr.DownloadString($url) | Out-String).length -as [INT]
             }
         Else {
             Write-Verbose "Downloading web page and displaying source code" 
             $wc.DownloadString($url)       
             }
         
         }
     Catch {
         Write-Warning "$($Error[0])"
         }
     }   
 }
`

