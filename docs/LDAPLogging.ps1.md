---
Author: patrick sczepanski
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3674
Published Date: 2015-10-01t13
Archived Date: 2015-08-16t00
---

# ldaplogging - 

## Description

functions to view/enable/disable ldap query logging on a dc and parse eventlog message from logged queries.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 function Private:Configure-Logging {
 <#
 	.SYNOPSIS
 		Enables, disables or view current LDAP Logging settings on a domain controller.
 
 	.DESCRIPTION
 		Enables, disables or view current LDAP Logging settings on a domain controller.
         
         Use one of the following aliases:
         
         Get-LDAPLogging
             View current LDAP logging settings
             
         Enable-LDAPLogging
             Enables LDAP logging. Logging is set to log every single LDAP query and stores it in directory services log.
 
         Disable-LDAPLogging
             Disables LDAP logging. Logging is set to its default values.
 
 	.EXAMPLE
 		PS C:\> Get-LDAPLogging
 
 	.EXAMPLE
 		PS C:\> Enable-LDAPLogging DC1
 
 	.INPUTS
 		N/A
 
 	.OUTPUTS
 		N/A
 
 	.NOTES
 		Author:    Patrick Sczepanski 
         Version:   1.0
         Email:     patrick -at- sczepanski -dot- com
                    patrick -dot- redtoo -at- redtoo -dot- com
         Blog:      http://redtoo.com/blog
         Copyright: 2012
 
 	.LINK
 		Get-LDAPLogging
         Enable-LDAPLogging
         Disable-LDAPLogging
         Get-LDAPEventLog
 
 #>
     
     Param (
         [string]
         $HostName = $env:COMPUTERNAME
     )
     
     [System.Nullable``1[[System.Int32]]]$EnableLogging = $null
     switch ( $MyInvocation.InvocationName ) {
         "Enable-LDAPLogging"  { 
             [System.Nullable``1[[System.Int32]]]$EnableLogging = $true
             [bool]$ReadWrite = $true
             [int]$Threshold = 1
             [int]$FieldEngDef = 5
             break
         }
         "Disable-LDAPLogging" { 
             [System.Nullable``1[[System.Int32]]]$EnableLogging = $false
             [bool]$ReadWrite = $true
             [int]$Threshold = 0
             [int]$FieldEngDef = 0
             break
         }
         Default     { 
             [System.Nullable``1[[System.Int32]]]$EnableLogging = $null
             [bool]$ReadWrite = $false
         }
     }
 
     if ( -not (Test-Connection $HostName -Quiet -Count 1) ) {
         Write-Warning "Cannot Ping $HostName"
         continue
     }
     if ( -not ( 
         Get-WmiObject -query "select producttype from Win32_OperatingSystem where producttype='2'" `
                       -ComputerName $HostName -ErrorAction SilentlyContinue) ) {
         Write-Warning "$HostName is not a domain controller or you do not have access."
         continue
     }
     
     $NTDSParams = "SYSTEM\CurrentControlSet\Services\NTDS\Parameters"
     $ExpThre = "Expensive Search Results Threshold"
     $IneThre = "Inefficient Search Results Threshold"
 
     $NTDSDiag = "SYSTEM\CurrentControlSet\Services\NTDS\Diagnostics"
     $FieldEng = "15 Field Engineering"
 
     $baseKey = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey( "LocalMachine", $HostName   )
 
     try {
         $key = $baseKey.OpenSubKey($NTDSParams,$ReadWrite)
         if ( $EnableLogging -eq $true ) { 
             $key.SetValue( $ExpThre, 1, [Microsoft.Win32.RegistryValueKind]::DWord )
             $key.SetValue( $IneThre, 1, [Microsoft.Win32.RegistryValueKind]::DWord )
         } elseif ( $EnableLogging -eq $false ) {
             $key.DeleteValue( $ExpThre )
             $key.DeleteValue( $IneThre )
         }
         $ExpThreValue = $key.GetValue($ExpThre)
         $IneThreValue = $key.GetValue($IneThre)
         $key.Close()
     }
     catch {
         Write-Warning "Failed to read or change $NTDSParams.`r`n$_"
     }
     try {
         $key = $baseKey.OpenSubKey($NTDSDiag,$ReadWrite)
         if ( $EnableLogging -ne $null ) { 
             $key.SetValue( $FieldEng, $FieldEngDef )
         } 
         $FieldEngValue = $key.GetValue($FieldEng)
         $key.Close()
     }
     catch {
         Write-Warning "Failed to read or change $NTDSDiag.`r`n$_"
     }
     $baseKey.Close()
     New-Object PSObject -Property @{
         "FieldEngineering" = $FieldEngValue
         "ExpensiveThreshold" = $ExpThreValue
         "InefficientThreshold" = $IneThreValue
     }
 }
 New-Alias Get-LDAPLogging Configure-Logging -Force
 New-Alias Enable-LDAPLogging Configure-Logging -Force
 New-Alias Disable-LDAPLogging Configure-Logging -Force
 
 Function Global:Get-LDAPEventLog {
 <#
 	.SYNOPSIS
 		Searches the Directory Service Eventlog for event ID 1644, parses the message text and returns a custom object.
 
 	.DESCRIPTION
 		Searches the Directory Service Eventlog for event ID 1644, parses the message text and returns a custom object.
         
         Use "Enable-LDAPLogging" to enable logging LDAP queries on a domain controller.
 
 	.EXAMPLE
 		PS C:\> Get-LDAPEventLog
 
 	.INPUTS
 		[string],[int]
 
 	.OUTPUTS
 		[LDAPLookups]
 
 	.NOTES
 		Author:    Patrick Sczepanski 
         Version:   1.0
         Email:     patrick -at- sczepanski -dot- com
                    patrick -dot- redtoo -at- redtoo -dot- com
         Blog:      http://redtoo.com/blog
         Copyright: 2012
 
 	.LINK
 		Get-LDAPLogging
         Enable-LDAPLogging
         Disable-LDAPLogging
 
 #>
 Param (
         [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Position=0)]
         [string]
         $DNSHostName = $env:COMPUTERNAME,
         
         [int]
         $LookupMinutes = 60
         
     )
     Begin {
         $Script:AlreadyLookedUp = @{}
 Add-Type @'
 using System;
         public class LDAPLookups {
             public string DNSHostName;
             public string ClientIP;
             public System.Nullable<int> ClientPort;
             public string ClientName;
             public string StartNode;
             public string Filter;
             public string SearchScope;
             public string Attributes;
             public string ServerControls;
             public System.Nullable<int> VisitedEntries;
             public System.Nullable<int> ReturnedEntries;
             public string Date;
             public string Time;
         }
 '@      
 
 $RegEx = @"
 (?msx)
 Client:\r\n
   (?<Client>.*)\:(?<Port>.*)\r\n?
 
 Starting\snode\:\r\n
   (?<StartNode>.*?)\r\n
 
 Filter:\r\n
   (?<Filter>.*?)\r\n
 
 Search\sscope:\r\n
   (?<SearchScope>.*?)\r\n
 
 Attribute\sselection:\r\n
   (?<Attributes>.*?)\r\n
 
 Server\scontrols:\r\n
   (?<ServerControls>.*?)\r\n
 
 Visited\sentries:\r\n
   (?<VisitedEntries>.*?)\r\n
 
 Returned\sentries:\r\n
   (?<ReturnedEntries>.*)
 "@
     }
     Process {
         Write-Host "... $DNSHostName ..."
         Get-WinEvent -ComputerName $DNSHostName  -FilterHashtable @{ "LogName"="Directory Service" ; "ID"=1644; StartTime= [datetime]::Now.AddMinutes( -$LookupMinutes ) } |
             Foreach-Object { 
                 if ( $_.Message -match $RegEx ) { 
                      New-Object LDAPLookups -Property @{
                         "DNSHostName" = $DNSHostName
                         "ClientIP" = $Matches.Client 
                         "ClientPort" = ($Matches.Port).Trim()
                         "ClientName" = $(   $ClientIP = $Matches.Client
                                             try {
                                                 if( -not $AlreadyLookedUp.contains( $ClientIP ) ){
                                                     $AlreadyLookedUp.$ClientIP = ( [System.Net.Dns]::GetHostByAddress( $ClientIP ).HostName )
                                                 }
                                             }
                                             catch {
                                                 $AlreadyLookedUp.$ClientIP = $ClientIP
                                             }
                                              $AlreadyLookedUp.$ClientIP
                                         )
                         "StartNode" = ($Matches.StartNode).Trim()
                         "Filter" = ($Matches.Filter).trim()
                         "SearchScope" = ($Matches.SearchScope).trim()
                         "Attributes" = ($Matches.Attributes).trim()
                         "ServerControls" = ($Matches.ServerControls).trim()
                         "VisitedEntries" = ($Matches.VisitedEntries).trim()
                         "ReturnedEntries" = ($Matches.ReturnedEntries).trim()
                         "Date" = $_.TimeCreated.ToString( "yyyy.MM.dd" )
                         "Time" = $_.TimeCreated.ToString( "HH:mm:ss" )
                     }
                 }
             } 
     } 
     End {
         Remove-Variable -Scope Script -Name AlreadyLookedUp
     }
 } 
 Write-Host "LDAPLogging functions loaded into Global."
`

