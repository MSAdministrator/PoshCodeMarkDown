---
Author: david retherford
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2992
Published Date: 2012-10-07t06
Archived Date: 2012-01-14t07
---

# update phone # in ad - 

## Description

strips off the ’1-’ prefix from phone numbers stored in active directory.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #------------------------------------------------------------------------------
 #------------------------------------------------------------------------------
 $data = @()
 $users = Get-QADUser -sizelimit 0 `
     | Where-Object { $_.PhoneNumber -like "1-*" } `
     | Where-Object { $_.PhoneNumber.Length -eq 14 } `
     | Sort-Object -Property Name
 foreach ( $user in $users )
 {
     if ( $user -eq $null )
     {
         continue
     }
     $newPhoneNumber = $user.PhoneNumber.ToString()
     $newPhoneNumber = $newPhoneNumber.Substring(2,12)
     $dataLine = New-Object psObject
     $dataLine | Add-Member -MemberType NoteProperty -Name "User" -Value $user.Name
     $dataLine | Add-Member -MemberType NoteProperty -Name "OldPhone" -Value $user.PhoneNumber
     $dataLine | Add-Member -MemberType NoteProperty -Name "NewPhone" -Value $newPhoneNumber
     $dataLine | Add-Member -MemberType NoteProperty -Name "DN" -Value $user.DN
     $data += $dataLine
     Set-QADUser -Identity $user.DN -PhoneNumber $newPhoneNumber 
 }
 if ( $data.Count -gt 0 )
 {
     $data | Out-GridView
 }
`

