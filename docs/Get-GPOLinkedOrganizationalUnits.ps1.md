---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6109
Published Date: 
Archived Date: 2016-03-18t21
---

#  - 

## Description

function to find organizational units linked to a given gpo

## Comments



## Usage



## TODO



## function

`get-gpolinkedorganizationalunits`

## Code

`#
 #
 <#
 .EXAMPLE  
     Get-GPO -Name TestOU | Get-GPOLinkedOrganizationalUnits
 #>
 Function Get-GPOLinkedOrganizationalUnits {
     param(
         [Parameter(ValueFromPipeline=$true, Mandatory=$true)][Microsoft.GroupPolicy.Gpo]$GPO
     )
 
     Get-ADOrganizationalUnit -Filter { LinkedGroupPolicyObjects -eq $gpo.Path }
 }
`

