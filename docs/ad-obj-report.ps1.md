---
Author: voytas
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4605
Published Date: 2013-11-12t19
Archived Date: 2013-11-14t15
---

# ad obj report - 

## Description

i would like group my ad obj report scripts to one file. i started with some functions about users and computers. i have a request to check my code to poit errors and what i could write better. thx for any tip and info!

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 
 
 
 if(!(get-module activedirectory)) {
 import-module activedirectory
 } 
 #}
 
 $inactivedays = 30
 $inactivedayscomp = 30
 $comptop = 20
 $userdname = "ou=users,dc=domain,dc=local"
 $compdname = "ou=comps,dc=domain,dc=local"
 
 
 function userinactive {
 write-host ;
 write-host "U¿ytkownicy aktywni i czasem nielogowania d³u¿szymy ni¿ $($inactivedays) dni" -ForegroundColor Magenta
 $date=(get-date).AddDays(-$inactivedays)
 $users=get-aduser `
     -Properties lastlogondate `
     -Filter {enabled -eq $true -and lastlogondate -lt $date -and samaccountname -notlike "*2*" -and samaccountname -notlike "*test*"} `
     -SearchBase $userdname | `
     sort lastlogondate
 $users | ft samaccountname, lastlogondate, distinguishedname -AutoSize
 write-host "Znalezionych u¿ytkowników: $(($users).count)" -ForegroundColor green
 write-host "---------------------------------------------"
 write-host ;
 }
 
 function userinactivenotloged {
 write-host ;
 write-host "U¿ytkownicy aktywni i nigdy niezalogowani" -ForegroundColor Magenta
 $users=get-aduser `
     -Properties lastlogondate, lastlogontimestamp,whencreated `
     -Filter {enabled -eq $true -and samaccountname -notlike "*2*" -and samaccountname -notlike "*test*"} `
     -SearchBase $userdname | `
     ? {(!$_.lastlogontimestamp -eq "*")}
 $users | sort samaccountname | ft samaccountname, distinguishedname, whencreated -AutoSize
 write-host "Znalezionych u¿ytkowników: $(($users).count)" -ForegroundColor green
 write-host "---------------------------------------------"
 write-host ;
 }
 
 
 
 function compinactive {
 write-host ;
 write-host "Komputery aktywne i czasem nielogowania d³u¿szymy ni¿ $($inactivedays) dni" -ForegroundColor Magenta
 $date=(get-date).AddDays(-$inactivedayscomp)
 $comps=get-adcomputer `
     -Properties lastlogondate `
     -Filter {enabled -eq $true -and lastlogondate -lt $date -and samaccountname -notlike "*template*" -and samaccountname -notlike "*pool*" -and samaccountname -notlike "*thinapp*"} `
     -SearchBase $compdname | `
     sort lastlogondate
 $comps | ft samaccountname, lastlogondate, distinguishedname -AutoSize
 write-host "Znalezionych komputerów: $(($comps).count)" -ForegroundColor green
 write-host "---------------------------------------------"
 write-host ;
 }
 
 function compinactivetop {
 write-host ;
 write-host "Komputery aktywne i czasem nielogowania d³u¿szymy ni¿ $($inactivedays) dni - TOP ($($comptop))" -ForegroundColor Magenta
 $date=(get-date).AddDays(-$inactivedayscomp)
 $comps=get-adcomputer `
     -Properties lastlogondate `
     -Filter {enabled -eq $true -and lastlogondate -lt $date -and samaccountname -notlike "*template*" -and samaccountname -notlike "*pool*" -and samaccountname -notlike "*thinapp*"} `
     -SearchBase $compdname | `
     sort lastlogondate
 $comps | `
     select-object -first $comptop | `
     ft samaccountname, lastlogondate, distinguishedname -AutoSize
 write-host "Top $($comptop) z komputerów: $(($comps).count)" -ForegroundColor green
 write-host "---------------------------------------------"
 write-host ;
 }
 
 
 
 cls
 userinactivenotloged
`

