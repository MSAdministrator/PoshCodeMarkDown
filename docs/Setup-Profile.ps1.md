---
Author: jmh6182
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5158
Published Date: 2014-05-10t03
Archived Date: 2014-05-15t01
---

# setup-profile - 

## Description

creates a blank profile

## Comments



## Usage



## TODO



## function

`setup-profile`

## Code

`#
 #
 Function Setup-Profile{
     
     $hasProfile = Test-Path -Path $profile
 
     if ($hasProfile -eq $false){
         $answer = Read-Host "No profile detected. Would you like to create one? (Y)es or (N)o"
         while("y","n","yes","no" -notcontains $answer)
         {
         	$answer = Read-Host "Yes or No"
         }
         
             if ($answer -eq "y"){
                 New-Item -Path $profile -ItemType "file" -Force
             } 
     }
 }
`

