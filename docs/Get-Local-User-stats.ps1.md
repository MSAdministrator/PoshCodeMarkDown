---
Author: michael wulff
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6045
Published Date: 2016-10-13t13
Archived Date: 2016-11-11t11
---

# get local user stats - 

## Description

get local user accounts info in forms of a csv file with the following attributes

## Comments



## Usage



## TODO



## script

`convert-userflag`

## Code

`#
 #
 $Workdir = 'C:\scripts\'
 $Computer = $env:COMPUTERNAME
 $ExcludedAccounts = '^Administrator|^Guest'
 $Output = $Workdir + 'Output.csv'
 
 
 Function Convert-UserFlag  {
 
   Param ($UserFlag)
 
   $List = New-Object System.Collections.ArrayList
 
   Switch  ($UserFlag) {
 
   ($UserFlag  -BOR 0x0001)  {[void]$List.Add('SCRIPT')}
   ($UserFlag  -BOR 0x0002)  {[void]$List.Add('ACCOUNTDISABLE')}
   ($UserFlag  -BOR 0x0008)  {[void]$List.Add('HOMEDIR_REQUIRED')}
   ($UserFlag  -BOR 0x0010)  {[void]$List.Add('LOCKOUT')}
   ($UserFlag  -BOR 0x0020)  {[void]$List.Add('PASSWD_NOTREQD')}
   ($UserFlag  -BOR 0x0040)  {[void]$List.Add('PASSWD_CANT_CHANGE')}
   ($UserFlag  -BOR 0x0080)  {[void]$List.Add('ENCRYPTED_TEXT_PWD_ALLOWED')}
   ($UserFlag  -BOR 0x0100)  {[void]$List.Add('TEMP_DUPLICATE_ACCOUNT')}
   ($UserFlag  -BOR 0x0200)  {[void]$List.Add('NORMAL_ACCOUNT')}
   ($UserFlag  -BOR 0x0800)  {[void]$List.Add('INTERDOMAIN_TRUST_ACCOUNT')}
   ($UserFlag  -BOR 0x1000)  {[void]$List.Add('WORKSTATION_TRUST_ACCOUNT')}
   ($UserFlag  -BOR 0x2000)  {[void]$List.Add('SERVER_TRUST_ACCOUNT')}
   ($UserFlag  -BOR 0x10000)  {[void]$List.Add('DONT_EXPIRE_PASSWORD')}
   ($UserFlag  -BOR 0x20000)  {[void]$List.Add('MNS_LOGON_ACCOUNT')}
   ($UserFlag  -BOR 0x40000)  {[void]$List.Add('SMARTCARD_REQUIRED')}
   ($UserFlag  -BOR 0x80000)  {[void]$List.Add('TRUSTED_FOR_DELEGATION')}
   ($UserFlag  -BOR 0x100000)  {[void]$List.Add('NOT_DELEGATED')}
   ($UserFlag  -BOR 0x200000)  {[void]$List.Add('USE_DES_KEY_ONLY')}
   ($UserFlag  -BOR 0x400000)  {[void]$List.Add('DONT_REQ_PREAUTH')}
   ($UserFlag  -BOR 0x800000)  {[void]$List.Add('PASSWORD_EXPIRED')}
   ($UserFlag  -BOR 0x1000000)  {[void]$List.Add('TRUSTED_TO_AUTH_FOR_DELEGATION')}
   ($UserFlag  -BOR 0x04000000)  {[void]$List.Add('PARTIAL_SECRETS_ACCOUNT')}
   }
 
   $List -join '; '
 
 }
 ([ADSI]"WinNT://$Computer").Children | ? {$_.SchemaClassName -eq 'user'} |  foreach {
           $groups = $_.Groups() | % {$_.GetType().InvokeMember("Name", 'GetProperty', $null, $_, $null)}
           $_ | Select @{n='Username';e={$_.Name}},
               @{n='FullName';e={$_.FullName}},
               @{n='LastUsed';e={$_.LastLogin}},
               @{n='GroupMembership';e={$groups -join ' ; '}},
               @{n='Description';e={$_.Description}},
               @{n='Status';e={Convert-UserFlag $_.Userflags.Value}}
 } | Export-Csv -NoTypeInformation -Encoding UTF8 -Delimiter ',' -Path $Output
`

