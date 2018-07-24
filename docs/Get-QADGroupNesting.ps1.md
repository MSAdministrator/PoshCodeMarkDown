---
Author: paul brice
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1890
Published Date: 
Archived Date: 2010-07-17t01
---

# get-qadgroupnesting - 

## Description

i came across an article on the “microsoft active directory powershell blog”, it has a great script for analysing nested group memberships. unfortunatly to use the powershell script you need to be running windows 2008 servers for the active directory cmdlets to be available. so not put off i converted the script to use the quest cmdlets available with the “quest active roles management” pssnapin. you will need to have these installed for the script to function.

## Comments



## Usage

ps c

## TODO



## function

`get-groupnesting`

## Code

`#
 #
 Param (
     [Parameter(Mandatory=$true,
         Position=0,
         ValueFromPipeline=$true,
         HelpMessage="DN or ObjectGUID of the AD Group."
     )]
     [string]$groupIdentity,
     [switch]$showTree
     )
 Add-PSSnapin -Name Quest.ActiveRoles.ADManagement -ErrorAction SilentlyContinue
 $global:numberOfRecursiveGroupMemberships = 0
 $lastGroupAtALevelFlags = @() 
 
 function Get-GroupNesting ([string] $identity, [int] $level, [hashtable] $groupsVisitedBeforeThisOne, [bool] $lastGroupOfTheLevel)
 {
     $group = $null
     $group = Get-QADGroup -Identity $identity -SizeLimit 0
     if($lastGroupAtALevelFlags.Count -le $level)
     {
         $lastGroupAtALevelFlags = $lastGroupAtALevelFlags + 0
     }
     if($group -ne $null)
     {
         if($showTree)
         {
             for($i = 0; $i -lt $level - 1; $i++)
 	    {
                 if($lastGroupAtALevelFlags[$i] -ne 0)
                 {
                     Write-Host -ForegroundColor Blue -NoNewline "  "
                 }
                 else
                 {
                 }
             }
             if($level -ne 0)
             {
                 if($lastGroupOfTheLevel)
                 {
                 }
                 else
                 {
                 }
             } 
             Write-Host -ForegroundColor Blue $group.Name
         }
         $groupsVisitedBeforeThisOne.Add($group.DN,$null)
         $global:numberOfRecursiveGroupMemberships ++
         $groupMemberShipCount = $group.memberOf.Count
         if ($groupMemberShipCount -gt 0)
         {
             $maxMemberGroupLevel = 0
             $count = 0
             foreach($groupDN in $group.memberOf)
             {
                 $count++
                 $lastGroupOfThisLevel = $false
                 if($count -eq $groupMemberShipCount){$lastGroupOfThisLevel = $true; $lastGroupAtALevelFlags[$level] = 1}
                 {
                     $memberGroupLevel = Get-GroupNesting -Identity $groupDN -Level $($level+1) -GroupsVisitedBeforeThisOne $groupsVisitedBeforeThisOne -lastGroupOfTheLevel $lastGroupOfThisLevel
                     if ($memberGroupLevel -gt $maxMemberGroupLevel){$maxMemberGroupLevel = $memberGroupLevel}
                 }
             }
             $level = $maxMemberGroupLevel
         }
         {
             return $level
         }
         return $level
     }
 }
 $global:numberOfRecursiveGroupMemberships = 0
 $groupObj = Get-QADGroup -Identity $groupIdentity -SizeLimit 0
 if($groupObj)
 {
     [int]$maxNestingLevel = Get-GroupNesting -Identity $groupIdentity -Level 0 -GroupsVisitedBeforeThisOne @{} -lastGroupOfTheLevel $false
 	Add-Member -InputObject $groupObj -MemberType NoteProperty  -Name MaxNestingLevel -Value $maxNestingLevel -Force
     Add-Member -InputObject $groupObj -MemberType NoteProperty  -Name NestedGroupMembershipCount -Value $($global:numberOfRecursiveGroupMemberships - 1) -Force
 	$groupObj | Select-Object Name,DN,MaxNestingLevel,NestedGroupMembershipCount | Format-List
 }
`

