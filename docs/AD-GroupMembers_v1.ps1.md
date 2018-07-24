---
Author: angel-keeper
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1738
Published Date: 
Archived Date: 2011-01-02t20
---

# ad-groupmembers_v1 - 

## Description

use this cmdlet to retrieve the directory objects that are members of a certain group in active directory.

## Comments



## Usage



## TODO



## function

`ad-groupmembers`

## Code

`#
 #
 function AD-GroupMembers() {
 param (
 $Domen,
 $Group,
 $User
 )
 if ($User){$Connection = Get-Credential -Credential $user}
 if($Connection){$Member = Get-QADGroupMember -Service $Domen -Identity $Group -Credential $Connection -SizeLimit 0 -ErrorAction SilentlyContinue | Sort Name | Format-Table Name,NTAccountName,Sid,AccountIsDisabled -AutoSize}
 else{$Member = Get-QADGroupMember -Service $Domen -Identity $Group -SizeLimit 0 -ErrorAction SilentlyContinue | Sort Name | Format-Table Name,NTAccountName,Sid,AccountIsDisabled -AutoSize}
 $Member
 }
`

