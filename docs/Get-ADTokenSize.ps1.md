---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6453
Published Date: 2017-07-22t10
Archived Date: 2017-04-17t14
---

# get-adtokensize - 

## Description

this advanced function will return information about the estimated token size and group memberships of the specified user.

## Comments



## Usage



## TODO



## function

`get-adtokensize`

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 function Get-ADTokenSize
 {
 
     <#
     .SYNOPSIS
     This cmdlets estimates the tokensize of a given user based on Active Directory group memberships.
 
     .DESCRIPTION
     This cmdlets estimates the tokensize of a given user based on Active Directory group memberships.
 
     It requires the Active Directory module to run.
 
     The returned token size value is only an estimation!
 
     .EXAMPLE
     Get-ADTokenSize -Identity JohnDoe
 
     Get the estimated token size of user JohnDoe
     .EXAMPLE
     Get-ADUser -Filter { GivenName -eq "John" } | Get-ADTokenSize
 
     Get the tokensize of all users named John.
 
     .PARAMETER Identity
     Specify the SamAccountName, DistinguishedName, objectGUID or SID of the user. Supports pipeline input.
 
     .PARAMETER Server
     Specifies the Active Directory Domain Services instance to connect to.
 
     #>
 
     [CmdletBinding()]
     param([Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
           [Alias('SamAccountName','DistinguishedName','ObjectGUID','SID')]
           [string] $Identity,
           [string] $Server = $env:USERDNSDOMAIN)
 
     BEGIN { }
 
     PROCESS {
 
         try {
             $UserDN = Get-ADUser -Identity $Identity -Server $Server -ErrorAction Stop | select -ExpandProperty DistinguishedName -ErrorAction Stop
         }
         catch {
             Write-Error "Failed to locate the user in Active Directory. The error was: $($Error[0])"
             return
         }
 
         try {
             $Groups = Get-ADGroup -LDAPFilter "(member:1.2.840.113556.1.4.1941:=$UserDN)" -Properties sIDHistory -Server $Server -ErrorAction Stop
         }
         catch {
             Write-Error "Failed to lookup group memberships. The error was: $($Error[0])"
             return
         }
 
         [int] $UserTokenSize = 1200
         [int] $GlobalGroups = 0
         [int] $UniversalGroups = 0
         [int] $DomainLocalGroups = 0
         [int] $GroupsWithSidHistory = 0
 
         foreach ($Group in $Groups) {
     
             Remove-Variable GroupTokenSize -ErrorAction SilentlyContinue
 
             if ($Group.SIDHistory.Count -ge 1) {
                 [int] $GroupTokenSize = 40
                 $GroupsWithSidHistory++
             }
             else {
                 [int] $GroupTokenSize = switch ($Group.GroupScope)
                                         {
                                         'Global'       {  8 ; $GlobalGroups++ }
                                         'Universal'    {  8 ; $UniversalGroups++ }
                                         'DomainLocal'  { 40 ; $DomainLocalGroups++ }
                                         }
             }
 
             $UserTokenSize += $GroupTokenSize
 
         }
 
         [int] $AllGroups = $GlobalGroups + $UniversalGroups + $DomainLocalGroups
 
         $returnObject = New-Object System.Object
         $returnObject | Add-Member -Type NoteProperty -Name DistinguishedName -Value $UserDN
         $returnObject | Add-Member -Type NoteProperty -Name EstimatedTokenSize -Value $UserTokenSize
         $returnObject | Add-Member -Type NoteProperty -Name GlobalGroups -Value $GlobalGroups
         $returnObject | Add-Member -Type NoteProperty -Name UniversalGroups -Value $UniversalGroups
         $returnObject | Add-Member -Type NoteProperty -Name DomainLocalGroups -Value $DomainLocalGroups
         $returnObject | Add-Member -Type NoteProperty -Name GroupsWithSidHistory -Value $GroupsWithSidHistory
         $returnObject | Add-Member -Type NoteProperty -Name AllGroups -Value $AllGroups
 
         Write-Output $returnObject
     }
 
     END { }
 }
`

