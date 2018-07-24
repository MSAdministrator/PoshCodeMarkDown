---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5176
Published Date: 2014-05-21t00
Archived Date: 2014-05-24t12
---

# llist context - 

## Description

emulates perl’s list context in powershell.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 <#
 
 After you execute Add-Type, use it like:
 
 [list()]$lhs = $rhs
 
 
 This allows you to override PowerShell's sometimes annoying behavior of:
 
 [object[]]$a = $null
 
 treating $null as $null instead of an empty array via:
 
 [object[]][list()]$a = $null
 
 NOTE: Order matters. PowerShell evaluates attributes in order from RIGHT to LEFT.
 
 #>
 
 
 Add-Type -TypeDefinition @'
 using System;
 using System.Management.Automation;
 
 [AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
 public class ListAttribute : ArgumentTransformationAttribute {
 	private static readonly object[] Value = new Object[0];
 
 	public override Object Transform(EngineIntrinsics engineIntrinsics, Object inputData) {
 		if (inputData == null || inputData == System.Management.Automation.Internal.AutomationNull.Value) {
 			return ListAttribute.Value;
 		}
 		return inputData;
 	}
 }
 '@
`

