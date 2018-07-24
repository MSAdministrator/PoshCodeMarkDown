---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2042
Published Date: 
Archived Date: 2010-08-06t18
---

# get-dcsfromdns - 

## Description

a function that allows me to query dns on my internal servers for domain controllers

## Comments



## Usage



## TODO



## function

`get-dcsfromdns`

## Code

`#
 #
 function Get-DCsFromDNS($DomainName){    
 $DCs = get-dns _ldap._tcp.dc._msdcs.$DomainName -Type srv | select -ExpandProperty RecordsRR | 
  %{$_.record.target} | select -Unique | sort | %{
 get-dns $_ | select -ExpandProperty Answers | select Name,@{n='IPAddress';e={$_.Record}}}
 return $DCs
 }
`

