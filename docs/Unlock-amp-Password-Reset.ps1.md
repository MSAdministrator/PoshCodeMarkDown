---
Author: matt schmitt
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: utf-8
License: cc0
PoshCode ID: 3814
Published Date: 2012-12-04t12
Archived Date: 2012-12-06t03
---

# unlock &amp; password reset - 

## Description

if this helps you, please tweet it!

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
   Author:   Matt Schmitt
   Date:     12/4/12 
   Version:  2.0 
   From:     USA 
   Email:    ithink2020@gmail.com 
   Website:  http://about.me/schmittmatt
   Twitter:  @MatthewASchmitt
   
   Description
   A script checking for Locked Account, checking where a user is locked out, unlocking the user's account and for resetting a user's password.  
   
   UPDATED 12/4/12
     Cleaned up Checking LockedOut Status code - replaced with foreach statement that looks at $Servers array
     Cleaned up Unlock code - replaced with foreach statement that looks at $Servers array
     Cleaned up Get pwdlastset date - rewrote to use the method I was using for other lookups for AD properties.
 
 #>
 
 
 Import-Module ActiveDirectory
 
 Write-Host ""
 Write-Host "PowerShell AD Password Tool"
 Write-Host ""
 Write-Host "This tool displays the Exparation Date of a user's Password and thier Lockedout"
 Write-Host "Status.  It will then allow you to unlock and/or reset the password."
 Write-Host ""
 
 Write-Host ""
 
 
 $servers = "AUSDC01.intranet.theknot.com", "AUSDC02.intranet.theknot.com", "AUSDC03.intranet.theknot.com", "CORPDC01.intranet.theknot.com", "LADC03.intranet.theknot.com", "NYCDC04.intranet.theknot.com", "NYCDC05.intranet.theknot.com", "omadc01.intranet.theknot.com", "omadc02.intranet.theknot.com", "REDDC02.intranet.theknot.com"
 
 
 
 $count = Search-ADAccount �LockedOut | where { $_.Name -ne "Administrator" -and $_.Name -ne "Guest" } | Measure-Object | Select-Object -expand Count
 
 
 If ( $count -gt 0 ) {
 
     Write-Host "Current Locked Out Accounts on your LOCAL Domain Controller:"
     Search-ADAccount �LockedOut | where { $_.Name -ne "Administrator" -and $_.Name -ne "Guest" } | Select-Object Name, @{Expression={$_.SamAccountName};Label="Username"},@{Expression={$_.physicalDeliveryOfficeName};Label="Office Location"},@{Expression={$_.TelephoneNumber};Label="Phone Number"},@{Expression={$_.LastLogonDate};Label="Last Logon Date"}  | Format-Table -AutoSize
     
 }else{
     
 
 }
 
 
 Write-Host ""
 
 
 $user = Read-Host "Enter username of the employee you would like to check or [ Ctrl+c ] to exit"
 
 cls 
 
 Write-Host ""
 Write-Host ""
 
 $Name = (Get-ADUser -Filter {samAccountName -eq $user } -Properties * | Select-Object -expand DisplayName)
 $phone = (Get-ADUser -Filter {samAccountName -eq $user } -Properties * | Select-Object -expand telephoneNumber)
 
 Write-Host "$Name's phone number is:  $phone"
 
 Write-Host ""
 Write-Host ""
 
 
 [datetime]$today = (get-date)
 
 
 $passdate2 = [datetime]::fromfiletime((Get-ADUser -Filter {samAccountName -eq $user } -Properties * | Select-Object -expand pwdlastset))
 
 
 
 
 
 $searcher=New-Object DirectoryServices.DirectorySearcher
 $searcher.Filter="(&(samaccountname=$user))"
 $results=$searcher.findone()
 $passdate = [datetime]::fromfiletime($results.properties.pwdlastset[0])
 
 
 Write-Host "passdate: $passdate"
 
 
 #>
 
 
 $PwdAge = ($today - $passdate2).Days
 
 If ($PwdAge -gt 90){
 
     Write-Host "Password for $user is EXPIRED!"
     Write-Host "Password for $user is  $PwdAge days old."
 
 }else{
 
     Write-Host "Password for $user is $PwdAge days old."
 
 }
 
 
 
 
 Write-Host ""
 Write-Host ""
 Write-Host "Checking LockedOut Status on U.S. Domain Controllers:"
 
 foreach ($object in $servers) {
 
     switch (Get-ADUser -server $object -Filter {samAccountName -eq $user } -Properties * | Select-Object -expand lockedout) { 
     
         "False" {"$object `t `t Not Locked"} 
         
         "True" {"$object `t `t LOCKED"}
    
     }
 }
 
 
 Write-Host ""
 Write-Host ""
 
 
 [int]$y = 0
 
 
 $option = Read-Host  "Would you like to (1) Unlock user, (2) Reset user's password, (3) Unlock and reset user's password or (4) Exit?"
 
 cls
 
 While ($y -eq 0) {
     
     switch ($option)
     {
         "1" { 
 
 
                 foreach ($object in $servers) {
                     
                     Write-Host "Unlocking account on $object"
                     Unlock-ADAccount -Identity $user -server $object
 
                 }         
                                 
                 
                 
                 $Lock = (Get-ADUser -Filter {samAccountName -eq $user } -Properties * | Select-Object -expand lockedout)
 
                 Write-Host ""
 
                 switch ($Lock)
                 {
                     "False" { Write-Host "$user is unlocked." }
                     "True" { Write-Host "$user is LOCKED Out." }
                 }                
                 
             
                 Write-Host ""
                 Write-Host "Press any key to Exit."
                 
                 $y += 5
                 
                 $x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
                 
             
             }
         "2" { 
                 
                 $newpass = (Read-Host -AsSecureString "Enter user's New Password")
                 
                 
                 Write-Host ""
                 Write-Host "Resetting Password on Local DC"
                 Write-Host ""
                 Set-ADAccountPassword -Identity $user -NewPassword $newpass
                 
                 Write-Host ""
                 Write-Host "Resetting Password on CORPDC01"
                 Write-Host ""
                 Set-ADAccountPassword -Server CORPDC01.intranet.theknot.com -Identity $user -NewPassword $newpass
                              
                            
                 Write-Host ""
                 Write-Host "Press any key to Exit."
                 $x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
                 
                 $y += 5
     
             }
         "3" {
     
                 $newpass = (Read-Host -AsSecureString "Enter user's New Password")
                 
                 Write-Host ""
                 Write-Host "Resetting Password on Local DC..."
                 Write-Host ""
                 Set-ADAccountPassword -Identity $user -NewPassword $newpass
                 
                 Write-Host ""
                 Write-Host "Resetting Password on CORPDC01 - for faster replication..."
                 Write-Host ""
                 Set-ADAccountPassword -Server CORPDC01.intranet.theknot.com -Identity $user -NewPassword $newpass
                 
                 Write-Host ""
                 Write-Host "Password for $user has been reset."
                 Write-Host ""
                 
                 
                 
                 foreach ($object in $servers) {
                     
                     Write-Host "Unlocking account on $object"
                     Unlock-ADAccount -Identity $user -server $object
 
                 }                
 
                 
                 $Lock = (Get-ADUser -Filter {samAccountName -eq $user } -Properties * | Select-Object -expand lockedout)
 
                 Write-Host ""
 
                 switch ($Lock)
                 {
                     "False" { Write-Host "$user is unlocked." }
                     "True" { Write-Host "$user is LOCKED Out." }
                 }                
                 
             
                 Write-Host ""
                 Write-Host "Press any key to Exit."
 
                 
                 $y += 5
                 
                 $x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
     
             }
         
         "4" {
     
                 $y += 5
     
             }
             
         default {
                 
                 Write-Host "You have entered and incorrect number."
                 Write-Host ""
                 $option = Read-Host  "Would you like to (1) Unlock user, (2) Reset user's password, (3) Unlock and reset user's password or (4) Exit?"
         
             }
     }
 }
`

