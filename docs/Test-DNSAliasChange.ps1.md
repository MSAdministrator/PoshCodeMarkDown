---
Author: ben newton
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6364
Published Date: 2016-06-01t14
Archived Date: 2016-06-04t16
---

# test-dnsaliaschange - 

## Description

a functional test for confirming dns alias changes have taken effect for users.

## Comments



## Usage



## TODO



## function

`test-dnsaliaschange`

## Code

`#
 #
 Function Test-DNSAliasChange{
 <#
     .Synopsis
     Tests to see if a DNS alias has changed - pings until timeout or it matches the new hostname.
 
     .Description
     Used as a test script. It runs until it wither times out or the DNS alias returns the correct name. Useful for automated deployments. 
     
     .Parameter NewComputerName 
     The Computer which you are moving the DNS alias to
 
     .Parameter OldComputerName
     The Computer you are moving the DNS alias from
 
     .Parameter DNSAlias
     The DNS alias you have changed
 
     .Parameter Timeout
     How long (in minutes) before you wish the script to end. 
 
     .Example
     Test-DNSAliasChange -NewComputerName MyNewBox -OldComputerName MyOldBox -
 #>
     [CmdletBinding(SupportsShouldProcess=$True)]
     Param(
     [Parameter(Mandatory=$True)]
     [string]
     $NewComputerName,
 
     [Parameter(Mandatory=$False)]
     [string]
     $OldComputerName,
 
     [Parameter(Mandatory=$True)]
     [string]
     $DNSAlias,
 
     [Parameter(Mandatory=$False)]
     [ValidateRange(0,[int]::MaxValue)]
     [int]
     $Timeout=1
     )
     $Time = [System.Diagnostics.Stopwatch]::StartNew()
 
     $Success = $False
 
     While($Time.Elapsed.Minutes -le $Timeout){
 
         $Obj = Test-Connection $DNSAlias -Count 1 
         $IP = $Obj.IPV4Address.IPAddressToString
         $Hostname = [net.dns]::GetHostByAddress($IP).HostName.split(".")[0]
 
         If($Hostname -match $NewComputerName){
             Write-Verbose "Change!"
             $Success = $True
             Break
         }
         ElseIf($Hostname -match $OldComputerName){
             Write-Verbose "No Change"
         }
         Else{
             Write-Warning "This DNS Alias does not refer to either HeadNode!"
         }
         sleep 10
     }
     $Time.Stop()
     $Final = $Time.ElapsedMilliseconds
     If($True){
         write-Verbose "Done in $Final MS"
     }
     Else{
         Write-Error "Script ended without success after $Final MS"
     }
     New-Object -TypeName psobject -Property @{NewComputerName=$NewComputerName;OldComputerName=$OldComputerName;DNSAlias=$DNSAlias;Success=$Success;ElapsedMS=$Final}
 }
`

