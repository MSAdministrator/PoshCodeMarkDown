---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2181
Published Date: 2011-09-09t21
Archived Date: 2016-05-17t17
---

# invoke-localizedscript.p - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 Set-StrictMode -Version Latest
 
 $messages = DATA {
     @{
         Greeting = "Hello, {0}"
         Goodbye = "So long."
     }
 }
 
 Import-LocalizedData messages -ErrorAction SilentlyContinue
 
 $messages.Greeting -f "World"
 $messages.Goodbye
`

