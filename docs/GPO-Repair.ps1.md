---
Author: jayneticmuffin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5692
Published Date: 2015-01-15t13
Archived Date: 2015-06-10t02
---

# gpo repair - 

## Description

i wanted a quick function that would tell me what policies were missing a specific group/account, and have the ability to resolve the problem. this function will display your query by default, and will apply the settings using the -apply parameter.  enjoy!

## Comments



## Usage



## TODO



## function

`fix-gpopermission`

## Code

`#
 #
 Function Fix-GPOPermission {
 <#
 	.SYNOPSIS
 		Repairs GPOs that have had an account removed from the delegation tab. Does NOT give GPOAPPLY rights.
 	.DESCRIPTION
         
 	.PARAMETER TargetName
 		Account that should be given access (ie. "Authenticated Users")
 	.PARAMETER TargetType
 		Valid names are "Computer", "Group", "User"
 	.PARAMETER Apply
 		Applies the settings
 	.PARAMETER ShowAll
 		Displays all the GPOs and appends [Correct] to those that the change will not apply to.
 	.EXAMPLE
 		PS C:\> Fix-GPOPermission -TargetName 'Authenticated Users' -TargetType 'Group'
 		Shows the group policies that do not have the queried account permissions set
 	.EXAMPLE
 		PS C:\> Fix-GPOPermission -TargetName 'Authenticated Users' -TargetType 'Group' -Apply
 		Shows the group policies that do not have the queried account permissions set, and applies the settings.
 		NOTE: If more rights are already set to a policy, then those being applied, there is no change to the policy.
 	.EXAMPLE
 		PS C:\> Fix-GPOPermission -TargetName 'Authenticated Users' -TargetType 'Group' -ShowAll
 #>
 	[cmdletbinding()]
 	param(
 		[Parameter(Mandatory=$true)][string]$TargetName,
 		[Parameter(Mandatory=$true)][ValidateSet("Computer","Group","User")][string]$TargetType,
 		[Switch]$Apply,
 		[Switch]$ShowAll
 	)
 	BEGIN {
 		$Domain = (Get-ADDomain).DNSRoot -as [string]
 		$gpos = Get-GPO -All -Domain $Domain | Sort DisplayName
 		Write-Verbose "  Found $($gpos.Count) total GPOs"
 		$counter = 0
 	}
 	PROCESS {
 		ForEach ($gpo in $gpos) {
 			Try { 
 				$null = Get-GPPermission -Guid $gpo.id -TargetName $TargetName -TargetType $TargetType -DomainName $Domain -ErrorAction 'Stop'
 				If ($ShowAll) {
 					$props = @{
 						'DisplayName'	= "[Correct] - $($gpo.DisplayName)";
 						'ID'		= $gpo.Id
 					}
 					$objGPO = New-Object -TypeName PSObject -Property $props
 					Write-Output $objGPO
 				}
 			}
 			Catch [System.Exception] {
 				$counter++
 				$props = @{
 					'DisplayName'	= $gpo.DisplayName;
 					'ID'		= $gpo.Id
 				}
 				$objGPO = New-Object -TypeName PSObject -Property $props
 				Write-Output $objGPO
 				If ($Apply) {
 					Set-GPPermission -Guid $gpo.Id -TargetName $user -TargetType Group -PermissionLevel:GpoRead
 				}
 			}
 		}
 		If ($counter -gt 0) {
 			Write-Warning "  Found $counter GPOs with '$TargetName' rights missing"
 		}
 	}
 	END {	}
 }
`

