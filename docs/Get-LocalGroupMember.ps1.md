---
Author: bsonposh
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2866
Published Date: 2012-07-28t10
Archived Date: 2017-01-30t09
---

# get-localgroupmember - 

## Description

gets local group memebers

## Comments



## Usage



## TODO



## function

`get-localgroupmember`

## Code

`#
 #
 function Get-LocalGroupMember
 {
     <#
         .Synopsis
             Get the local group membership.
 
         .Description
             Get the local group membership.
 
         .Parameter ComputerName
             Name of the Computer to get group members (Default localhost.)
 
         .Parameter Group
             Name of the group to get members from (Default Administrators.)
 
         .Example
             Get-LocalGroupMember
             Description
             -----------
             Get Administrators group Members for the localhost
 
         .Example
             Get-LocalGroupMember -Computer MyPC -group User
             Description
             -----------
             Get Members of Users Group from MyPC
 
         .Example
             $Servers | Get-LocalGroupMember -group "Remote Desktop Users"
             Description
             -----------
             Get Members of Users Group from "Remote Desktop Users" on all $Servers
 
         .OUTPUTS
             PSCustomObject
 
         .INPUTS
             System.String
 
         .Link
             N/A
 
         .Notes
             NAME:      Get-LocalGroupMember
             AUTHOR:    YetiCentral\bshell
             Website:   www.bsonposh.com
             LASTEDIT:  10/13/2009 18:25:15
     #>
 
     [Cmdletbinding()]
     Param(
         [alias('dnsHostName')]
         [Parameter(ValueFromPipelineByPropertyName=$true,ValueFromPipeline=$true)]
         [string]$ComputerName = $Env:COMPUTERNAME,
 
         [Parameter()]
         [string]$Group= "Administrators"
     )
 begin {
 
 
         Write-Verbose " [Get-LocalGroupMember] :: Start Begin"
         function GetGroupMembers($Server,$Group)
         {
             Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: `$Server = $Server"
             Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: `$Group = $Group"
 
             $GroupObject = Get-WMIObject win32_group -filter "LocalAccount='$true' AND Name='$Group'" -computername $Server -
 
             $Query = "SELECT * FROM Win32_GroupUser WHERE GroupComponent = `"Win32_Group.Domain='$Server',Name='$Group'`""
             Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: `$Query = $Query"
             $GroupUsers = get-wmiobject -Query $Query -computername $Server  -ea STOP
 
             Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: Processing Group Members "
 
             if($GroupUsers)
             {
                 foreach($User in $GroupUsers)
                 {
                     Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: `$User.PartComponent = $($User.PartComponent
 
                     $RegEx = '\\\\{0}\\root\\cimv2\:Win32_.*\="(?<Domain>.*)",Name="(?<User>.*)"' -f $Server
                     Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: RegEx = $RegEx "
                     $User.PartComponent -match $Regex | out-null
 
                     Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: Creating Custom Object "
                     $myobj = @{}
                     Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: `$myobj.ComputerName = $Server"
                     $myobj.ComputerName = $Server
                     Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: `$myobj.UserName = $($matches.User)"
                     $myobj.UserName = $matches.User
                     Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: `$myobj.Domain = $($matches.Domain)"
                     $myobj.Domain = $matches.Domain
                     Write-Verbose " [Get-LocalGroupMember] :: GetGroupMembers :: `$myobj.Group = $Group"
                     $myobj.Group = $Group
                     $obj = New-Object PSObject -Property $myobj
                     $obj.PSTypeNames.Clear()
                     $obj.PSTypeNames.Add('BSonPosh.LocalGroup')
                     $obj
                 }
             }
         }
 
         Write-Verbose " [Get-LocalGroupMember] :: End Begin"
 
 
 }
 process {
 
 
         Write-Verbose " [Get-LocalGroupMember] :: Start Process"
         if($ComputerName -match "(.*)(\$)$")
         {
             $ComputerName = $ComputerName -replace "(.*)(\$)$",'$1'
         }
         if(Test-Host $ComputerName -TCPPort 135)
         {
             Write-Verbose " [Get-LocalGroupMember] :: Processing Computer: $ComputerName"
             try
             {
                 GetGroupMembers $ComputerName $Group
             }
             catch
             {
                 Write-Host " Host [$ComputerName] Failed with Error: $($Error[0])" -ForegroundColor Red
             }
         }
         else
         {
             Write-Host " Host [$ComputerName] Failed Connectivity Test " -ForegroundColor Red
         }
         Write-Verbose " [Get-LocalGroupMember] :: End Process"
 
 
 }
 }
`

