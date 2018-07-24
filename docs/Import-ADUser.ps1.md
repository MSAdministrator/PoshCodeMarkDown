---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5072
Published Date: 2016-04-11t18
Archived Date: 2016-03-18t21
---

# import-aduser.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #############################################################################
 ##
 ##
 ##
 #############################################################################
 
 <#
 
 .SYNOPSIS
 
 Create users in Active Directory from the content of a CSV.
 
 .DESCRIPTION
 
 In the user CSV, One column must be named "CN" for the user name.
 All other columns represent properties in Active Directory for that user.
 
 For example:
 CN,userPrincipalName,displayName,manager
 MyerKen,Ken.Myer@fabrikam.com,Ken Myer,
 DoeJane,Jane.Doe@fabrikam.com,Jane Doe,"CN=MyerKen,OU=West,OU=Sales,DC=..."
 SmithRobin,Robin.Smith@fabrikam.com,Robin Smith,"CN=MyerKen,OU=West,OU=..."
 
 .EXAMPLE
 
 PS >$container = "LDAP://localhost:389/ou=West,ou=Sales,dc=Fabrikam,dc=COM"
 PS >Import-ADUser.ps1 $container .\users.csv
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $Container,
 
     [Parameter(Mandatory = $true)]
     $Path
 )
 
 Set-StrictMode -Off
 
 $userContainer = [adsi] $container
 
 if(-not $userContainer.Name)
 {
     Write-Error "Could not connect to $container"
     return
 }
 
 $users = @(Import-Csv $Path)
 if($users.Count -eq 0)
 {
     return
 }
 
 foreach($user in $users)
 {
     $username = $user.CN
     $newUser = $userContainer.Create("User", "CN=$username")
 
     foreach($property in $user.PsObject.Properties)
     {
         if($property.Name -eq "CN")
         {
             continue
         }
 
         if(-not $property.Value)
         {
             continue
         }
 
         $newUser.Put($property.Name, $property.Value)
     }
 
     $newUser.SetInfo()
 }
`

