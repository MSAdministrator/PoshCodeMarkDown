---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5870
Published Date: 2016-05-25t18
Archived Date: 2016-05-17t14
---

# add-poshendpointaccess - 

## Description

function add an account/group to a winrm-endpoint, by default the default powershell endpoint. see comment based help or this blog post for more information

## Comments



## Usage



## TODO



## function

`add-poshendpointaccess`

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 function Add-PoShEndpointAccess
 {
     <#
     .Synopsis
        Adds a group or user to a PowerShell (WinRM) endpoint to allow remote management.
 
     .DESCRIPTION
        This function will edit the SDDL of a PowerShell (WinRM) endpoint to 
        allow remote management for the specified account/group.
 
        If you run this against a remote computer, CredSSP needs to be enabled and you need
        to restart the WinRM-service manually afterwards (this function uses WinRM to connect
        to the remote machine, which is why it will not restart the service itself).
 
     .PARAMETER SamAccountName
        The SamAccount name of the user or group that you want to give access to. Could also be in the form
        domain\SamAccountName, for example contoso\Administrator.
 
     .PARAMETER ComputerName
        Specifies the computer on which the command runs. The default is the local computer.
 
     .PARAMETER EndpointName
        Specifies then name of the WinRM endpoint you want to configure, the default is Microsoft.PowerShell.
 
     .EXAMPLE
        Add-PoShEndpointAccess -SamAccountName "contoso\PoShUsers" -ComputerName MyPoShEndpoint.contoso.com
 
     #>
 
     [CmdletBinding()]
     Param
     (
         [Parameter(Mandatory=$true,
                    ValueFromPipelineByPropertyName=$true)]
         $SamAccountName,
 
         [Parameter(Mandatory=$false)]
         $ComputerName = '.',
 
         [Parameter(Mandatory=$false)]
         $EndpointName = 'Microsoft.PowerShell'
     )
 
     Begin { }
 
     Process {
         if ($ComputerName -eq '.' -OR $ComputerName -eq "$($env:COMPUTERNAME)") {
                 $IdentityObject = New-Object Security.Principal.NTAccount $SamAccountName
                 try {
                     $sid = $IdentityObject.Translate([Security.Principal.SecurityIdentifier]).Value
                 }
                 catch {
                     throw "Failed to translate $SamAccountName to a valid SID."
                 }
 
                 try {
                     $PSSConfig = Get-PSSessionConfiguration -Name $EndpointName -ErrorAction Stop
                 }
                 catch {
                     if ($_.Tostring() -like '*access is denied*') {
                         throw 'You need to have Admin-access to run this command!'
                     }
                 }
 
                 $existingSDDL = $PSSConfig.SecurityDescriptorSDDL
                 $isContainer = $false
                 $isDS = $false
 
                 $SecurityDescriptor = New-Object -TypeName Security.AccessControl.CommonSecurityDescriptor -ArgumentList $isContainer,$isDS, $existingSDDL
                 $accessType = 'Allow'
                 $accessMask = 268435456
                 $inheritanceFlags = 'none'
                 $propagationFlags = 'none'
                 $SecurityDescriptor.DiscretionaryAcl.AddAccess($accessType,$sid,$accessMask,$inheritanceFlags,$propagationFlags)
 
                 $null = Set-PSSessionConfiguration -Name $EndpointName -SecurityDescriptorSddl ($SecurityDescriptor.GetSddlForm('All')) -Confirm:$false -Force
 
         }
         else {
             Invoke-Command -ArgumentList $SamAccountName,$EndpointName -ScriptBlock {
                 $IdentityObject = New-Object Security.Principal.NTAccount $args[0]
                 $EndpointName = $args[1]
 
                 try {
                     $sid = $IdentityObject.Translate([Security.Principal.SecurityIdentifier]).Value
                 }
                 catch {
                     throw "Failed to translate $($args[0]) to a valid SID."
                 }
 
                 try {
                     $PSSConfig = Get-PSSessionConfiguration -Name $EndpointName -ErrorAction Stop
                 }
                 catch {
                     if ($_.Tostring() -like '*access is denied*') {
                         throw 'You need to have Admin-access and enable CredSSP to run this command remotely!'
                     }
                 }
 
                 $existingSDDL = $PSSConfig.SecurityDescriptorSDDL
                 $isContainer = $false
                 $isDS = $false
 
                 $SecurityDescriptor = New-Object -TypeName Security.AccessControl.CommonSecurityDescriptor -ArgumentList $isContainer,$isDS, $existingSDDL
                 $accessType = 'Allow'
                 $accessMask = 268435456
                 $inheritanceFlags = 'none'
                 $propagationFlags = 'none'
                 $SecurityDescriptor.DiscretionaryAcl.AddAccess($accessType,$sid,$accessMask,$inheritanceFlags,$propagationFlags)
 
                 $null = Set-PSSessionConfiguration -Name $EndpointName -SecurityDescriptorSddl ($SecurityDescriptor.GetSddlForm('All')) -Confirm:$false -Force -NoServiceRestart
 
             } -ComputerName $ComputerName
         }
     }
 
     End { }
 }
`

