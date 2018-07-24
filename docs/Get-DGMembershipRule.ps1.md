---
Author: vidrine
Publisher: 
Copyright: 
Email: 
Version: 2014.04.07
Encoding: ascii
License: cc0
PoshCode ID: 5140
Published Date: 2014-05-01t19
Archived Date: 2014-05-05t02
---

# get-dgmembershiprule - 

## Description

query a dynamic group configured in quest activeroles and return an array of each membership rule.

## Comments



## Usage



## TODO



## function

`get-dgmembershiprule`

## Code

`#
 #
 function Get-DGMembershipRule {
 
     param (
         $Identity
     )
 
     $group = Get-QADGroup -Identity $Identity
     $strGroupDN = $group.DN
 
     $objGroup = $null
     $objGroup = [ADSI] "EDMS://$strGroupDN"
 
     $objRuleCollection = $objGroup.MembershipRuleCollection
 
     $results = @()
     forEach ( $rule in $objRuleCollection ) {
 
         $results += $rule
     }
 
     return $results
 
 <#
 .SYNOPSIS
    Query a dynamic group configured in Quest ActiveRoles and return an array of each membership rule.
 
 .DESCRIPTION
    Required:
    + Quest Active Roles cmdlets
    + Quest Active Roles Management Shell
 
    Query a dynamic group configured in Quest ActiveRoles and return an array of each membership rule.
 
    Rule _Type_ Codes:
    [1]  Include By Query
    [2]  Exclude By Query
    [3]  Include Explicitly
    [4]
    [5]
    [6]
 
 .NOTES
    Author  : Vidrine
    Date    : 2014.04.07
    Contact : vidrine.dev@gmail.com
 
 .PARAMETER Identity
    Specify the Identity of the target group.  Primarily, this will be the distinguishedName or
    sAMAccountName for the group object.
 
 .EXAMPLE
    Get-DGMembershipRule -Identity $groupName
    
    Base       : EDMS://OU=foo,OU=com
    Filter     : %displays_the_LDAP_query%
    Type       : %see_description_for_list_of_values%
    BaseGuid   : 
    IsModified :
 #>
 }
`

