---
Author: dethompson71
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4640
Published Date: 2016-11-25t22
Archived Date: 2016-10-21t07
---

# set-keepalivetime.ps1 - 

## Description

when moving very large mailboxes, the connection can time out.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
        .SYNOPSIS
               Change the setting for TCPIP KeepAliveTime on a server or several
               servers.
       
        .DESCRIPTION
               Change the setting for TCPIP KeepAliveTime on a server or several
               servers.
              
               When moving mailboxes with large number of folders or very large number
               of items it can take a very long time to enumerate the items. Sometimes
               it takes over 2 hours and the connection times out.
              
               We set the TCOPIP KeepAliveTime to it's max for the move or we reset
               it back to normal after a move is complete.
       
        .PARAMETER Name
               A collection of servers to set, reset, or read the KeepAliveTime
               registry setting.
              
        .PARAMETER KeepAliveTime
               The value to set KeepAliveTime. (A decimal number can be used.)
              
               Default is '0x000dbba0' - 15 Minutes
              
               0x000dbba0 ( 900000 decimal) 15 mnutes
               0x006ddd00 (7200000 decimal) is the installed default setting of 2 hours
              
              
        .PARAMETER ReadOnly
               Read the KeepAliveTime setting for each server in the collection of
               Servers
 	      (should be a "get" but I am too lazy to write another script.)
  
        .PARAMETER Hours
               Set the collection of servers to TCPIP KeepAliveTime of <N> hours.
              
  
        .PARAMETER Max
               Set the collection of servers to the maximum TCPIP KeepAliveTime setting.
               Defined as 0xffffffff - essentially "Forever"
                           
        .EXAMPLE
               $Servers = Get-MailboxServer | ?{$_.AdminDisplayVersion -match "^Version 14"}
               Set-KeepAliveTime -Name $Servers
              
               Sets the KeepAliveTime on all the Mailbox servers to the Default setting.
              
        .EXAMPLE
               $servers = get-mailboxserver | ?{$_.AdminDisplayVersion -match "^Version 14"}
               Set-KeepAliveTime -Name $Servers -SetMax
              
               Sets the KeepAliveTime on all the Mailbox servers to the Maximum setting.
              
        .EXAMPLE
               $servers = Get-ExchangeServer |?{$_.ServerRole -ne "Mailbox" -and $_.AdminDisplayVersion -match "^Version 14"}
               Set-KeepAliveTime -Name $Servers -Reset
              
               Sets the KeepAliveTime on all the Exchamge 2010 Servers servers (CAS, HUB,
               Mailbox)to the reset setting. Which can be different from default.
              
               Suppose you find a setting of 4 hours is fine for moving large mailboxes.
               Change the script to
               The 21600000 for 6 hours
       
       
        .NOTES
        This script requires PSRemoteRegistry
        http://psremoteregistry.codeplex.com/releases/view/65928
 	   
        Explaination for TCPIP settings:
        http://technet.microsoft.com/en-us/library/dd349797(v=WS.10).aspx
       
        Microsoft recomends is 5 Minutes for KeepAliveTime (2/17/2010)
        HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Tcpip\Parameters\
        KeepAliveTime | Reg_DWORD | 0x000493e0 (300000) 
       
        Symantec recomends 15 minutes (July 15, 2011)
        HKEY_LOCAL_MACHINE\System\CurrentControlSet\Services\Tcpip\Parameters\
        KeepAliveTime | Reg_DWORD | 0x000dbba0 (900000) 
       
        We use Symantic's recomendation as our default.
       
 #>
  
 [CmdletBinding()]
  
 Param (
        [Parameter(Position = 0, Mandatory = $true,ValueFromPipeLineByPropertyName=$true)]
        [String[]]
        $Name,
        [Parameter(Position = 1, Mandatory = $false)]
        [String]
        $KeepAliveTime = '0x000dbba0',
        [Parameter(Position = 3, Mandatory = $false)]
        [ValidateRange(1,24)]
        [Int]
        $Hours = 0,
        [Parameter(Position = 2, Mandatory = $false)]
        [Switch]
        $ReadOnly,
        [Parameter(Position = 4, Mandatory = $false)]
        [Switch]
        $Max
 )
  
 if($(Hostname) -match '<server>') {
        Import-Module PSRemoteRegistry
 } Else {
        Write-Warning "You must run this on <server> to work."
        Exit
 }
  
 $Key = 'System\CurrentControlSet\Services\Tcpip\Parameters\'
 $Value = 'KeepAliveTime'
  
  
 if ($Name){
        ForEach ($Server in $Name) {
               if (Test-Connection -Count 1 -Quiet -ComputerName $Server){
                      if ($ReadOnly){
                            Write-Verbose "Checking Server: $($Server)"
                            Get-RegValue -ComputerName $Server -Key $key -Value $Value
                      } ElseIf ($Max){
                            $KeepAliveTime = '0xffffffff'
                            Write-Verbose "Set Max Server: $($Server)"
                            Set-RegDWord -ComputerName $Server -Key $Key -Value $Value -Data $KeepAliveTime -Force
                      } Else {
                            If ($Hours -gt 0){
                                   $KeepAliveTime = 3600000*$Hours
                                   Write-Verbose "Set Server: $($Server) to $($Hours) hours"
                                   Set-RegDWord -ComputerName $Server -Key $Key -Value $Value -Data $KeepAliveTime -Force
                            } Else {
                                   Set-RegDWord -ComputerName $Server -Key $Key -Value $Value -Data $KeepAliveTime -Force
                            }
                      }
               }  Else {
                            Write-Warning "Server $($Server) is not responding to pings."
               }
        }
 } Else {
        Write-Warning "Could not find any servers to process."
 }
`

