---
Author: erik mccarty
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 682
Published Date: 
Archived Date: 2009-01-05t16
---

# set-usercannotchangepass - 

## Description

set the "user cannot change password" property on an active

## Comments



## Usage



## TODO



## script

`set-usercannotchangepassword`

## Code

`#
 #
 #
 #
 #
 #
 #
 #
 #
 #
 function set-UserCannotChangePassword( [string] $sAMAccountName ){
    $everyOne = [System.Security.Principal.SecurityIdentifier]'S-1-1-0'
    $self = [System.Security.Principal.SecurityIdentifier]'S-1-5-10'
    $SelfDeny = new-object System.DirectoryServices.ActiveDirectoryAccessRule (
                               $self,'ExtendedRight','Deny','ab721a53-1e2f-11d0-9819-00aa0040529b')
    $SelfAllow = new-object System.DirectoryServices.ActiveDirectoryAccessRule (
                               $self,'ExtendedRight','Allow','ab721a53-1e2f-11d0-9819-00aa0040529b')
    $EveryoneDeny = new-object System.DirectoryServices.ActiveDirectoryAccessRule (
                            $Everyone,'ExtendedRight','Deny','ab721a53-1e2f-11d0-9819-00aa0040529b')
    $EveryOneAllow = new-object System.DirectoryServices.ActiveDirectoryAccessRule (
                            $Everyone,'ExtendedRight','Allow','ab721a53-1e2f-11d0-9819-00aa0040529b')
 
    $searcher = New-Object DirectoryServices.DirectorySearcher
    $searcher.filter = "(&(samaccountname=$sAMAccountName))"
    $results = $searcher.findone()
    $user = $results.getdirectoryentry()
 
    $user.psbase.get_ObjectSecurity().AddAccessRule($selfDeny)
    $user.psbase.get_ObjectSecurity().AddAccessRule($EveryoneDeny)
    $user.psbase.CommitChanges()
 }
`

