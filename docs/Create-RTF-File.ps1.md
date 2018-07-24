---
Author: sean kearney
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6761
Published Date: 2017-02-28t21
Archived Date: 2017-05-24t16
---

# create rtf file - 

## Description

very simple script to create an rtf (rich text format) file with windows powershell with variable substitution.  yes this could be a very basic mail merge type document without the use of microsoft word

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 param (
 [string]$Filename
 )
 #
 
 #
 #
 
 
 $Firstname="John"
 $Lastname="Smith"
 
 
 $Accountname="CONTOSO\\jsmith"
 $Password="LousyPass123"
 
 
 $Header+="{\rtf1\ansi\ansicpg1252\deff0\nouicompat\deflang1033{\fonttbl{\f0\fnil\fcharset0 Consolas;}}`r`n"
 $Header+="{\*\generator Riched20 6.2.8102}\viewkind4\uc1 `r`n"
 $Header+="\pard\sl276\slmult1\f0\fs22\lang9 \par`r`n"
 
 
 $Message+="Hello $Firstname $Lastname and Welcome to ABC\par`r`n"
 $Message+="Corporation.\par`r`n"
 $Message+="\par`r`n"
 $Message+="Your User ID is $Accountname\par`r`n"
 $Message+="Your Temporary Password is $Password\par`r`n"
 $Message+="\par`r`n"
 $Message+="Do not share this information and remember,\par`r`n"
 $Message+="We are watching....\par`r`n"
 $Message+="`r`n"
 
 
 $Footer="}`r`n"
 
 
 $Content=$Header+$Message+$Footer
 
 
 ADD-CONTENT -path $Filename -value $Content -force
`

