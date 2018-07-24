---
Author: jeremy d pavleck
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4831
Published Date: 2014-01-22t15
Archived Date: 2014-01-29t05
---

# get-o365 - 

## Description

just a simple function to show how to query office 365 via powershell

## Comments



## Usage



## TODO



## function

`get-o365`

## Code

`#
 #
 Function Get-O365 {
 $cred = Get-Credential -Message "Your username is your email address for O365"
 $session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri "https://ps.outlook.com/powershell/" -Credential $cred -Authentication Basic -AllowRedirection
 Import-PSSession $session
 $myMB = Get-Mailbox
 
 Write-Host -ForegroundColor "Green" "Hello $($myMB.Name), your email address is $($myMB.WindowsEmailAddress), and your account was created $($myMB.WhenCreated)"
 }
`

