---
Author: carsten kr
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4861
Published Date: 2014-01-30t18
Archived Date: 2014-02-09t11
---

# get-localadministrators - 

## Description

retrieves local administrators of a maschine using system.directoryservices.accountmanagement via well-known security identifiers

## Comments



## Usage



## TODO



## class

`get-localadministrators`

## Code

`#
 #
 <#
 
 .NOTES
 
     Author: Carsten Krüger - cakruege+poshcode@gmail.com
 
 #>
 
 Add-Type @'
 public class MyAccounts
 {
     public System.Collections.ArrayList users; 
     public System.Collections.ArrayList groups;
 }
 '@   
 
 function get-localadministrators {
     param ([string]$computername=$env:computername)
 
     $computername = $computername.toupper()
     
                 Add-Type -AssemblyName System.DirectoryServices.AccountManagement
                 $PrincipalContext = New-Object System.DirectoryServices.AccountManagement.PrincipalContext([System.DirectoryServices.AccountManagement.ContextType]::Machine, $computername)
                            
                 $GroupPrincipal = New-Object System.DirectoryServices.AccountManagement.GroupPrincipal($PrincipalContext)
                 $Searcher = New-Object System.DirectoryServices.AccountManagement.PrincipalSearcher
                 $Searcher.QueryFilter = $GroupPrincipal
                                               
                 $users = New-Object System.Collections.ArrayList
                 $groups = New-Object System.Collections.ArrayList
                 
                 $objOutput= New-Object MyAccounts
                               
                 foreach ($ladmin in $localadmins.Members)
                 {                 
                      if ($ladmin.ContextType -eq 'Machine')
                      {
                            [void] $users.Add($ladmin.Context.Name+'\'+$ladmin.SamAccountName)
                      }                                        
                     if ($ladmin.StructuralObjectClass -eq 'user') {
                       [void] $users.Add($ladmin.Context.Name+'\'+$ladmin.SamAccountName)
                     }                  
                     if ($ladmin.StructuralObjectClass -eq 'group') {
                       [void] $groups.Add($ladmin.Context.Name+'\'+$ladmin.SamAccountName)
                     }                                        
                 }    
                 
                 $objOutput.users=$users
                 $objOutput.groups=$groups
                 
                     
     return $objoutput
`

