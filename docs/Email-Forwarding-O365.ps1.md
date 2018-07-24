---
Author: matt schmitt
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: utf-8
License: cc0
PoshCode ID: 3801
Published Date: 2012-11-29t05
Archived Date: 2012-12-04t07
---

# email forwarding - o365 - 

## Description

script for forwarding and unforwarding email in office 365. this script makes doing the tasks easier, without having to use owa.  it saves time by not have to go into owa to forward an email, when i get the request.  i’m new at powershell (taking my first class next week), so i know there might be better ways of doing this.  however, this works for me and i think it i did pretty good for just starting out.

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
   Date:     11/28/12 
   Version:  1.0 
   From:     USA 
   Email:    ithink2020@gmail.com 
   Website:  http://about.me/schmittmatt
   Twitter:  @MatthewASchmitt
   
   Description
   A script for forwarding and unforwarding email for users in Office 365.  
 #>
 
 Write-Host ""
 Write-Host ""
 Write-Host "PowerShell Forward / Unforward Email Tool"
 Write-Host ""
 Write-Host ""
 Write-Host "Connecting to the Office 365 Exchange PowerShell Session..."
 Write-Host ""
 Write-Host ""
 Write-Host ""
 
 
 
 $cred = Get-Credential
 $Session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://ps.outlook.com/powershell -Credential $cred -Authentication Basic �AllowRedirection
 Set-ExecutionPolicy remotesigned -Force
 Import-PSSession $Session
 
 cls
 
 Write-Host ""
 Write-Host "PowerShell Forward / Unforward Email Tool"
 Write-Host ""
 Write-Host "This is used for forwarding and unforwarding other user's email."
 Write-Host ""
 Write-Host ""
 
 
 Write-Host "Would you like to:"
 Write-Host ""
 Write-Host "    (1) Forward an Email"
 Write-Host "    (2) Unforward an Email"
 Write-Host "    (3) Exit"
 Write-Host ""
 
 $todo = Read-Host "Please enter selection."
 
 [int]$checker = 0
 [int]$checker2 = 0
 
 While ($checker -eq 0) {
     
     Switch ($todo) {
         
         "1" {
                 
                 cls
                 
                 Write-Host ""
                 Write-Host ""
                 
                 Write-Host "You have selected to Forward a user's email."
                 
                 Write-Host ""
                 Write-Host ""
                 
                 $user = Read-Host "Enter username of the email account that needs to be forwarded" 
                 Write-Host ""
                 
                 $email = Read-Host "Enter the email address, where the user's email should be forwared to"
                 Write-Host ""
                 
                 $copy = Read-Host "Would the user like a copy left in thier mailbox? (Y)es or (N)o?"
                 
                 While ($checker2 -eq 0) {
                     
                     Switch ($copy) {
                         
                         "Y" {
                                 [bool]$deliver = $True
                                 
                                 $checker2 += 1
                             }
                         
                         "N" {
                                 [bool]$deliver = $False
                                 
                                 $checker2 += 1
                             }
                         
                         default  {  
                                     cls
                 
                                     Write-Host "You have entered and incorrect Option."
                                     Write-Host ""
                                     
                                     $copy = Read-Host "Would the user like a copy left in thier mailbox? (Y)es or (N)o?"
                                  }
                 }                   
                                                               
             }
 
                 
                 
                 Write-Host "Now Forwarding $user's email to $email..."
                 
                 Set-Mailbox $user �ForwardingSmtpAddress $email �DeliverToMailboxAndForward $deliver                            
                 
                 Remove-PSSession $Session
                 
                 Write-Host ""
                 Write-Host "If no errors, email has been Forwarded.  If there is an error, review the error and try again."
                 Write-Host ""
                 Write-Host "Press any key to Exit."
                 
                 $checker += 1
                 
                 $x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
         
             }
         "2" {
                 
                 cls
                 
                 Write-Host ""
                 Write-Host ""
                 
                 Write-Host "You have selected to Unforward a user's email."
                 Write-Host ""
                 Write-Host ""
                 
                 $user = Read-Host "Enter username of the email account that needs to be forwarded" 
                 Write-Host ""
                 
                 Write-Host "Now UnForwarding $user's email."
                 
                 Set-Mailbox $user �ForwardingSmtpAddress $null                          
                 
                 Remove-PSSession $Session
                 
                 Write-Host ""
                 Write-Host "If no errors, email has been UnForwarded.  If there is an error, review the error and try again."
                 Write-Host ""
                 Write-Host "Press any key to Exit."
                 
                 $checker += 1
                 
                 $x = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
         
             }
         "3" {
                 
                 Remove-PSSession $Session
                 
                 $checker += 1
         
             }
             
         default {
                 
                 cls
                 
                 Write-Host "You have entered and incorrect selection.  Please enter number corresponding to your selection."
                 Write-Host ""
                 Write-Host "Would you like to:"
                 Write-Host ""
                 Write-Host "    (1) Forward an Email"
                 Write-Host "    (2) Unforward an Email"
                 Write-Host "    (3) Exit"
                 Write-Host ""
                 
                 $todo = Read-Host "Please enter selection."
         
             }
     }
 }
`

