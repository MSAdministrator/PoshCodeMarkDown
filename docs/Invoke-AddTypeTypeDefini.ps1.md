---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2173
Published Date: 2011-09-09t21
Archived Date: 2016-03-19t00
---

# invoke-addtypetypedefini - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 #############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Demonstrates the use of the -TypeDefinition parameter of the Add-Type
 cmdlet.
 
 #>
 
 Set-StrictMode -Version Latest
 
 $newType = @'
 using System;
 
 namespace PowerShellCookbook
 {
     public class AddTypeTypeDefinitionDemo
     {
         public string SayHello(string name)
         {
             string result = String.Format("Hello {0}", name);
             return result;
         }
     }
 }
 
 '@
 
 Add-Type -TypeDefinition $newType
 
 $greeter = New-Object PowerShellCookbook.AddTypeTypeDefinitionDemo
 $greeter.SayHello("World");
`

