---
Author: jbandy
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1648
Published Date: 2010-02-18t11
Archived Date: 2017-03-18t12
---

# poshboard inactive ad ac - 

## Description

poshboard ad searcher showing stale computer accounts. borrowed parts from powergui powerpack

## Comments



## Usage



## TODO



## function

`get-domaincomputeraccounts`

## Code

`#
 #
 function Get-DomainComputerAccounts
 {
 
 $searcher = new-object DirectoryServices.DirectorySearcher([ADSI]"")
 
 $searcher.filter = "(&(objectClass=computer))"
 
 $searcher.CacheResults = $true
 $searcher.SearchScope = �Subtree�
 $searcher.PageSize = 1000
 
 $accounts = $searcher.FindAll()
 
 if($accounts.Count -gt 0)
 {
 foreach($account in $accounts)
 	{
 $pwdlastset = $account.Properties["pwdlastset"];
 
 $lastchange = [datetime]::FromFileTimeUTC($pwdlastset[0]);
 
 $datediff = new-TimeSpan $lastchange $(Get-Date);
 
 $obj = new-Object PSObject;
 
 $obj | Add-Member NoteProperty ComputerName($account.Properties["name"][0]);
 $obj | Add-Member NoteProperty LastPasswordChange($lastchange);
 $obj | Add-Member NoteProperty DaysSinceChange($datediff.Days);
 
 Write-Output $obj;
 }
 }
 }
 
 Get-DomainComputerAccounts | Where-Object {$_.DaysSinceChange -gt 30} | sort dayssincechange | Export-CSV C:\temp\AD.csv -NoType
 
 
 $dash = new-pbdashboard -Name "Object Password Age Method" -MaxColumns 1
 $exclude = "UnixServer01","NetworkAppliance","Server01","Server02","Server03"
 add-pbitem $dash (import-csv c:\temp\AD.csv | where {$exclude -notcontains $_.ComputerName} | out-pbdatagrid ComputerName,DaysSinceChange,LastPasswordChange -name "Computer accounts older than 30 days" -fontsize 10 -showgrouppane 0)
 $dash
`

