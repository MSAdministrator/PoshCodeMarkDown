---
Author: a monkey
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2604
Published Date: 2012-04-06t10
Archived Date: 2012-12-29t04
---

# sntp - 

## Description

get and set functions for sntp

## Comments



## Usage



## TODO



## script

`get-sntpserver`

## Code

`#
 #
 
 <#
 .SYNOPSIS
 Gets the SNTP servers a machine is configured to use
  
 .DESCRIPTION
 Gets the SNTP servers a machine is configured to use
  
 .PARAMETER Server
 The machine to get the SNTP Servers for
  
 .EXAMPLE
 PS C:\> Get-SNTPServer -Server MachineName
 This is return the SNTP servers for MachienName
  
 .EXAMPLE
 PS C:\> Get-SNTPServer -server Machine1 | Set-SNTPServer -server Machine2
 This will get the sntp server for Machine1 and set Machine2 to the same
 
 .EXAMPLE
 PS C:\> Get-Content servers.txt | Get-SNTPServer 
  
 .NOTES
     NAME        :  Get-SNTPServer
     VERSION     :  1.0
     LAST UPDATED:  04/04/2011
     AUTHOR      :  a Monkey
  
 .LINK
 http://amonkeysden.tumblr.com/post/4393059808/sntp-module
  
 #>
 function Get-SNTPServer
 {
     [cmdletbinding()]
     Param(
         [Parameter(
             Mandatory = $True,
             Position = 0,
             ParameterSetName = '',
             ValueFromPipeline = $True)]
         [string]$Server
     )
     [string] $z = net time \\$server /QUERYSNTP
     if (($z.contains("The current SNTP value is:"))) {
         #$out = "{0}: {1}" -f $Server,$z.split(" ")[5]
         Write-Output $z.split(" ")[5]
     } else {
         Write-Error "$z"
     }
 }
  
 <#
 .SYNOPSIS
 Sets the SNTP servers a machine is configured to use
  
 .DESCRIPTION
 Sets the SNTP servers a machine is configured to use
  
 .PARAMETER Servers
 The machines to set the SNTP Servers for
  
 .PARAMETER $SNTPServerList
 The machine to get the SNTP Servers for
  
 .EXAMPLE
 PS C:\> Set-SNTPServer -Server MachineName -SNTPServerList "dc1,dc2"
 This will set the SNTP servers to DC1 and DC2 for MachienName
  
 .EXAMPLE
 PS C:\> Get-SNTPServer -server Machine1 | Set-SNTPServer -server Machine2
 This will get the sntp server for Machine1 and set Machine2 to the same
 
 .EXAMPLE
 PS C:\> Get-Content servers.txt | Set-SNTPServer -SNTPServerList "dc1,dc2"
 
 .NOTES
     NAME        :  Set-SNTPServer
     VERSION     :  1.0
     LAST UPDATED:  04/04/2011
     AUTHOR      :  a Monkey
  
 .LINK
 http://amonkeysden.tumblr.com/post/4393059808/sntp-module
 
 #>
 function Set-SNTPServer
 {
     [cmdletbinding()]
     Param(
         [Parameter(
             Mandatory = $True,
             Position = 0,
             ParameterSetName = '',
             ValueFromPipeline = $True)]
         [string]$Server,
         [Parameter(
             Mandatory = $True,
             Position = 1,
             ParameterSetName = '',
             ValueFromPipeline = $True)]
         [string]$SNTPServerList
     )
     foreach ($s in $Server.Split(",")) 
     {
         [string] $z = net time \\$s /SETSNTP:$SNTPServerList
         if (($z.contains("The command completed successfully."))) {
             Write-Output $z
         } else {
             Write-Error $z
         }
     }
 }
  
  
 Export-ModuleMember Set-SNTPServer
 Export-ModuleMember Get-SNTPServer
`

