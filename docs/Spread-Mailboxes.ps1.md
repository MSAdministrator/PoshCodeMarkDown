---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jer@powershell.no
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1839
Published Date: 2010-05-13t17
Archived Date: 2016-06-01t12
---

# spread-mailboxes.ps1 - spread-mailboxes.ps1

## Description

script to spread mailboxes alphabetically across mailboxdatabases based on the first character in the user`s displayname.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################
 #
 #
 #
 #
 #
 #
 ###########################################################################
 
 Add-PSSnapin Microsoft.Exchange.Management.PowerShell.E2010 -ErrorAction SilentlyContinue
 
 
 foreach ($mailbox in (Get-Mailbox -ResultSize unlimited)) {
 
 $displayname = $mailbox.Displayname
 
 switch -regex ($displayname.substring(0,1)) 
     { 
        "[A-F]" {$mailboxdatabase = "MDB A-F"} 
        "[G-L]" {$mailboxdatabase = "MDB G-L"} 
        "[M-R]" {$mailboxdatabase = "MDB M-R"} 
        "[S-X]" {$mailboxdatabase = "MDB S-X" } 
        "[Y-Z]" {$mailboxdatabase = "MDB Y-Z" } 
         default {$mailboxdatabase = "MDB Y-Z" }
     }
  
 if (($mailbox.database.name) -ne $mailboxdatabase) {New-MoveRequest -Identity $mailbox -TargetDatabase $mailboxdatabase;Write-Host "Moving $displayname to $mailboxdatabase" -ForegroundColor Green}
  
 }
`

