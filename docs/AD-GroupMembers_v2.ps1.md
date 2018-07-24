---
Author: angel-keeper
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1739
Published Date: 
Archived Date: 2011-01-01t11
---

# ad-groupmembers_v2 - 

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
 if($Connection){$Member = Get-QADGroupMember -Service $Domen -Identity $Group -Credential $Connection -SizeLimit 0 -ErrorAction SilentlyContinue | Sort Name | Format-List Name,NTAccountName,Sid,AccountIsDisabled -AutoSize}
 else{$Member = Get-QADGroupMember -Service $Domen -Identity $Group -SizeLimit 0 -ErrorAction SilentlyContinue | Sort Name | Format-List Name,NTAccountName,Sid,AccountIsDisabled -AutoSize}
 $Member
 }
`

