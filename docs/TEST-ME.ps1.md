---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4056
Published Date: 
Archived Date: 2013-03-31t12
---

#  - 

## Description

test me

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 Import-Module ActiveDirectory -ErrorAction SilentlyContinue
 
 $defpassword = (ConvertTo-SecureString "pass@word1" -AsPlainText -force)
 
 $dnsroot = '@' + (Get-ADDomain).dnsroot
 
 $users = Import-Csv .\users.csv
 
 foreach ($user in $users) {
             {
                 try {
                     New-ADUser -SamAccountName $user.SamAccountName -Name ($user.FirstName + " " + $user.LastName) `
                     -DisplayName ($user.FirstName + " " + $user.LastName) -GivenName $user.FirstName -Surname $user.LastName `
                     -EmailAddress ($user.SamAccountName + $dnsroot) -UserPrincipalName ($user.SamAccountName + $dnsroot) `
                     -Title $user.title -Enabled $true -ChangePasswordAtLogon $false -PasswordNeverExpires  $true `
                     -AccountPassword $defpassword -PassThru `
                     }
                 catch [System.Object]
                     {
                         Write-Output "Could not create user $($user.SamAccountName), $_"
                     }
             }
             else
              {
                 try {
                     New-ADUser -SamAccountName $user.SamAccountName -Name ($user.FirstName + " " + $user.LastName) `
                     -DisplayName ($user.FirstName + " " + $user.LastName) -GivenName $user.FirstName -Surname $user.LastName `
                     -EmailAddress ($user.SamAccountName + $dnsroot) -UserPrincipalName ($user.SamAccountName + $dnsroot) `
                     -Title $user.title -manager $user.manager `
                     -Enabled $true -ChangePasswordAtLogon $false -PasswordNeverExpires  $true `
                     -AccountPassword $defpassword -PassThru `
                     }
                 catch [System.Object]
                     {
                         Write-Output "Could not create user $($user.SamAccountName), $_"
                     }
              }
         $filename = "$($user.SamAccountName).jpg"
         Write-Output $filename
 
         if (test-path -path $filename)
             {
                 Write-Output "Found picture for $($user.SamAccountName)"
 
                 $photo = [byte[]](Get-Content $filename -Encoding byte)
                 Set-ADUser $($user.SamAccountName) -Replace @{thumbnailPhoto=$photo} 
             }
    }
`

