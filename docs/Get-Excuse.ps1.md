---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2380
Published Date: 
Archived Date: 2010-11-30t16
---

# get-excuse - 

## Description

suffering from monday morning due to server failure. maybe an excuse should be automatically added to mails sent from members of my team.

## Comments



## Usage



## TODO



## function

`get-excuse`

## Code

`#
 #
 Function Get-Excuse {
   <#
     .Synopsis
       Retrieves a BOFH excuse from Jeff Ballard's website (http://jeffballard.us/).
     .Description
       Uses System.Net.WebRequest to retrieve page content as a string from the predefined URL. By default,
       Get-Excuse uses the system defined Proxy Server. Alternate proxy settings can be defined using the
       optional parameters.
     .Parameter ProxyServer
       A proxy server for this connection, in the format http://proxyserver:port.
     .Parameter Credential
       Alternate credentials to use for the proxy server.
     .Example
       Get-Excuse -Proxy "http://1.2.3.4:8080"
       Request an excuse using the specified proxy server.
   #>
 
   [CmdLetBinding()]
   Param(
     [ValidatePattern("https?://.+")]
     [Uri]$ProxyServer,
 
     [Management.Automation.PsCredential]$Credential
   )
 
   $WebRequest = [Net.WebRequest]::Create("http://pages.cs.wisc.edu/~ballard/bofh/bofhserver.pl")
   
 
   If ($ProxyServer) {
     $WebProxy = New-Object Net.WebProxy
     $WebProxy.Address = $ProxyServer
 
 
     If ($Credential) {
       $WebProxy.Credentials = $Credential.GetNetworkCredential()
     }
   }
 
   $Response = $WebRequest.GetResponse()
   If ($Response.StatusCode -eq "OK") {
 
     $ResponseStream = New-Object IO.StreamReader($Response.GetResponseStream(), "UTF8")
     $Page = $ResponseStream.ReadToEnd()
 
     $Page -Match '(?<=(<br><font size = "\+2">)).+' | Out-Null
     Return $Matches[0]
   }
   $Response.Close()
 
   Return ""
 }
 
 $Excuse = Get-Excuse
 If ($Excuse) {
   $Predicate = Get-TransportRulePredicate FromMemberOf
   $Predicate.Addresses = Get-DistributionGroup "address@domain.com"
 
   $Action = Get-TransportRuleAction SetHeader
   $Action.MessageHeader = "X-BOFH-Excuse"
   $Action.HeaderValue = $Excuse
 
   New-TransportRule -Name "BOFH Excuse" -Actions $Action -Comments "Includes the excuse of the day in an email header" `
     -Conditions $Predicate -Enabled $True -WhatIf
 }
`

