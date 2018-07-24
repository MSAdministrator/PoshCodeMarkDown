---
Author: dotps1
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6026
Published Date: 2016-09-19t17
Archived Date: 2016-05-17t13
---

# get-lastloggedonuser - 

## Description

gets the last logged on user of a workstation.

## Comments



## Usage



## TODO



## script

`get-lastloggedonuser`

## Code

`#
 #
 
 <#
 .SYNOPSIS
     Gets the last logged on user of a system.
 .DESCRIPTION
     Queries the Win32_UserProfile Windows Management Instrimentation Class and returns the result with the greatest LastUseTime value.
 .INPUTS
     System.String.
     System.Management.Automation.PSCredential.
 .OUTPUTS
     System.Management.Automation.PSCustomObject.
 .PARAMETER Name
     Name of the system.
 .PARAMETER Credential
     Account with administrative actions on the system.
     Uses current user context credentials by default.
 .EXAMPLE
     Get-LastLoggedOnUser
 
     UserName  LastLoggedOnTimeStamp CurrentlyLoggedOn
     --------  --------------------- -----------------
     MyPC\User 1/1/2015 12:00:00 AM               True
 .EXAMPLE
     Get-LastLoggedOnUser -Name 'Computer1','Computer2'
 
     UserName       LastLoggedOnTimeStamp CurrentlyLoggedOn
     --------       --------------------- -----------------
     Computer1\User 1/1/2015 12:00:00 AM               True
     Computer2\User 1/1/2015 12:00:00 AM              False
 .EXAMPLE
     Get-ADComputer -Filter { Name -like 'Contoso*' } | Get-LastLoggedOnUser -Credential (Get-Credential)
 
     UserName      LastLoggedOnTimeStamp CurrentlyLoggedOn
     --------      --------------------- -----------------
     Contoso1\User 1/1/2015 12:00:00 AM               True
     Contoso2\User 1/1/2015 12:00:00 AM              False
 .NOTES
     Administrative rights is required to use the function.
 .LINK
     http://dotps1.github.io
 #>
 
 Function Get-LastLoggedOnUser {
     
     [OutputType([PSCustomObject])]
 
     Param (
         [Parameter(
             ValueFromPipeline = $true,
             ValueFromPipelineByPropertyName = $true
         )]
         [ValidateScript({
             if (Test-Connection -ComputerName $_ -Count 1 -ErrorAction SilentlyContinue){
                 $true
             } else {
                 throw "Unable to contact $_."
             }
         })]
         [Alias(
             'ComputerName'
         )]
         [String[]]
         $Name = $env:COMPUTERNAME,
 
         [Parameter(
             ValueFromPipelineByPropertyname = $true
         )]
         [PSCredential]
         $Credential
     )
 
     Process {
         foreach ($n in $Name) {
             $params = @{
                 Class = 'Win32_UserProfile'
                 Namespace = 'root\CimV2'
                 ComputerName = $n
                 ErrorAction = 'Stop'       
             }
 
             if ($Credential) {
                 $params.Add('Credential', $Credential)
             }
             
             Get-WmiObject @params | Sort-Object -Property LastUseTime | ForEach-Object {
                 if (-not ($_.Special)) {
                     [PSCustomObject]@{
                         UserName = ([System.Security.Principal.SecurityIdentifier]$_.SID).Translate([System.Security.Principal.NTAccount]).Value
                         LastLoggedOnTimeStamp = $_.ConvertToDateTime($_.LastUseTime)
                         CurrentlyLoggedOn = $_.Loaded
                     }
                 }
             } | Select-Object -Last 1
         }
     }
 }
`

