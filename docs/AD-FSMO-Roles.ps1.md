---
Author: mike f robbins
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4104
Published Date: 2013-04-13t14
Archived Date: 2013-04-15t06
---

# ad fsmo roles - 

## Description

this powershell function returns a list of the active directory fsmo role holders for one or more active directory domains and forests. this function depends on the active directory module, specifically the get-addomain and get-adforest cmdlets. this module can either be installed locally, via the remote server administration tools or imported via implicit remoting prior to running this function. more information about this function can be found on my blog site

## Comments



## Usage



## TODO



## function

`get-fsmorole`

## Code

`#
 #
 function Get-FSMORole {
 <#
 .SYNOPSIS
 Retrieves the FSMO role holders from one or more Active Directory domains and forests.
 .DESCRIPTION
 Get-FSMORole uses the Get-ADDomain and Get-ADForest Active Directory cmdlets to determine
 which domain controller currently holds each of the Active Directory FSMO roles.
 .PARAMETER DomainName
 One or more Active Directory domain names.
 .EXAMPLE
 Get-Content domainnames.txt | Get-FSMORole
 .EXAMPLE
 Get-FSMORole -DomainName domain1, domain2
 #>
     [CmdletBinding()]
     param(
         [Parameter(ValueFromPipeline=$True)]
         [string[]]$DomainName = $env:USERDOMAIN
     )
     BEGIN {
         Import-Module ActiveDirectory -Cmdlet Get-ADDomain, Get-ADForest -ErrorAction SilentlyContinue
     }
     PROCESS {
         foreach ($domain in $DomainName) {
             Write-Verbose "Querying $domain"
             Try {
             $problem = $false
             $addomain = Get-ADDomain -Identity $domain -ErrorAction Stop
             } Catch { $problem = $true
             Write-Warning $_.Exception.Message
             }
             if (-not $problem) {
                 $adforest = Get-ADForest -Identity (($addomain).forest)
 
                 New-Object PSObject -Property @{
                     InfrastructureMaster = $addomain.InfrastructureMaster
                     PDCEmulator = $addomain.PDCEmulator
                     RIDMaster = $addomain.RIDMaster
                     DomainNamingMaster = $adforest.DomainNamingMaster
                     SchemaMaster = $adforest.SchemaMaster
                 }
             }
         }
     }
 }
`

