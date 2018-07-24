---
Author: oisin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 857
Published Date: 
Archived Date: 2009-02-10t03
---

# poc csharp expressions - 

## Description

proof of concept of running csharp expressions in powershell v2. very basic, no error checking.

## Comments



## Usage



## TODO



## function

`run-csharpexpression`

## Code

`#
 #
 function run-csharpexpression([string] $expression )
 {
 $global:ccounter = [int]$ccounter + 1
 $local:name  =  [system.guid]::NewGuid().tostring().replace('-','_').insert(0,"csharpexpr")
 $local:ns = "ShellTools.DynamicCSharpExpression.N${ccounter}"
 
 $local:template = @"
 using System;
 using System.IO;
 using System.Collections.Generic;
 using System.Linq;
 
 namespace $ns
 {
     public static class Runner
     {
         public static object RunExpression()
         {
             return [[EXPRESSION]];
         }
     }
 }
 "@
 
 $local:source = $local:template.replace("[[EXPRESSION]]",$expression)
 
 add-Type $local:source -Language CsharpVersion3 | out-Null
  invoke-Expression ('[' + $local:ns + '.Runner]::RunExpression()')
 }
 set-alias c run-csharpexpression 
 c DateTime.Now
 c "new{a=1,b=2,c=3}"
 c 'from x in Directory.GetFiles(@"c:\downloads") where x.Contains("win") select x'
`

