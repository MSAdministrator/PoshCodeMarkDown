---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 926
Published Date: 
Archived Date: 2009-03-15t01
---

# get-gender.ps1 - 

## Description

this script serves three purposes, including the obvious

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
    
    
    if($name.Length -lt 2) { throw "You need at least two letters in the name" }
    $name = "$($name[0])".ToUpper()  + $name.SubString(1).ToLower()
 
    switch(
       Invoke-Http GET "http://www.babynameaddicts.com/cgi-bin/search.pl" @{
          gender="ALL";searchfield="Names";origins="ALL";searchtype="matching";searchtext=$name
       } | Receive-Http Text "//font[b/font/text()='$name']/@color" )
    { 
       "fucshia" { "Femenine" }
    }
 }
 
 
`

