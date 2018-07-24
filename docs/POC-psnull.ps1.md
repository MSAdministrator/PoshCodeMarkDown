---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 893
Published Date: 
Archived Date: 2009-02-25t08
---

# poc $psnull - 

## Description

powershell converts all string types to strings, including nulls which end up as empty strings. use this if you need to pass a true null as a string to a dotnet api

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $a = @"
 
 using System;
 namespace ClassLibrary1
 {
 public static class Class1
 {
 public static string Foo(string value)
 {
 if (value == null)
     return "Special processing of null.";
 else
     return "'" + value + "' has been processed.";
 }
 }
 }
 "@
 
 $b = @"
 using System;
 namespace testnullD
 {
 public class nulltest
 {
 public nulltest(){}
 public override string ToString() {return null;}
 }
 }
 "@
 add-type $a
 add-type $b
 $psnull = new-object testnullD.nulltest
 add-type $a
 [ClassLibrary1.Class1]::Foo($null)
 [ClassLibrary1.Class1]::Foo($psnull)
`

