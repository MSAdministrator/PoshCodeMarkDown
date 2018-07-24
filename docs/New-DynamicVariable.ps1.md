---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2198
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# new-dynamicvariable.ps1 - 

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
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Creates a variable that supports scripted actions for its getter and setter
 
 .EXAMPLE
 
 PS >.\New-DynamicVariable GLOBAL:WindowTitle `
      -Getter { $host.UI.RawUI.WindowTitle } `
      -Setter { $host.UI.RawUI.WindowTitle = $args[0] }
 
 PS >$windowTitle
 Administrator: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
 PS >$windowTitle = "Test"
 PS >$windowTitle
 Test
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $Name,
 
     [Parameter(Mandatory = $true)]
     [ScriptBlock] $Getter,
 
     [ScriptBlock] $Setter
 )
 
 Set-StrictMode -Version Latest
 
 Add-Type @"
 using System;
 using System.Collections.ObjectModel;
 using System.Management.Automation;
 
 namespace Lee.Holmes
 {
     public class DynamicVariable : PSVariable
     {
         public DynamicVariable(
             string name,
             ScriptBlock scriptGetter,
             ScriptBlock scriptSetter)
                 : base(name, null, ScopedItemOptions.AllScope)
         {
             getter = scriptGetter;
             setter = scriptSetter;
         }
         private ScriptBlock getter;
         private ScriptBlock setter;
 
         public override object Value
         {
             get
             {
                 if(getter != null)
                 {
                     Collection<PSObject> results = getter.Invoke();
                     if(results.Count == 1)
                     {
                         return results[0];
                     }
                     else
                     {
                         PSObject[] returnResults =
                             new PSObject[results.Count];
                         results.CopyTo(returnResults, 0);
                         return returnResults;
                     }
                 }
                 else { return null; }
             }
             set
             {
                 if(setter != null) { setter.Invoke(value); }
             }
         }
     }
 }
 "@
 
 if(Test-Path variable:\$name)
 {
     Remove-Item variable:\$name -Force
 }
 
 $executioncontext.SessionState.PSVariable.Set(
     (New-Object Lee.Holmes.DynamicVariable $name,$getter,$setter))
`

