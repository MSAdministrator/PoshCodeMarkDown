---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5178
Published Date: 2014-05-22t01
Archived Date: 2014-05-24t12
---

# dict context - 

## Description

powershell has numeric and string contexts

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 <#
 
 [dict()]$lhs = $rhs
 
 Is basically equivalent to this pseudo-code:
 
 $lhs = $(if ($null -eq $rhs) {
 	@{}
 } elseif ($rhs -is [System.Collections.IDictionary] -or $rhs -is [System.Collections.Generic.IDictionary`2] -or $rhs -is [System.Collections.Generic.IReadOnlyDictionary`2])) {
 	$rhs
 } else {
 	throw ([ArgumentException]"Value is not a dictionary type")
 })
 
 
 Or in plain English, [dict()] converts $null to @{}, passes any [IDictionary]-like types as is, and fails otherwise.
 
 NOTES:
 PowerShell doesn't consider [IReadOnlyDictionary`2]s to be dictionaries, this code does.
 PowerShell doesn't consider [IDictionary`2]s that don't implement [IDictionary] to be dictionaries, this code does.
 
 
 #>
 
 Add-Type -TypeDefinition @'
 using System;
 using System.Management.Automation;
 
 [AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
 public class DictAttribute : ArgumentTransformationAttribute {
 	public override Object Transform(EngineIntrinsics engineIntrinsics, Object inputData) {
 		Object o = inputData;
 		PSObject pso = inputData as PSObject;
 		if (pso != null) {
 			o = (pso != System.Management.Automation.Internal.AutomationNull.Value) ? pso.BaseObject : null;
 		}
 		if (o == null)
 			return new System.Collections.Hashtable();
 		if (o is System.Collections.IDictionary)
 			return inputData;
 		foreach (Type t in o.GetType().GetInterfaces()) {
 			if (t.IsGenericType) {
 				Type g = t.GetGenericTypeDefinition();
 				if (g == typeof(System.Collections.Generic.IDictionary<,>) ||
 				    g == typeof(System.Collections.Generic.IReadOnlyDictionary<,>))
 				{
 					return inputData;
 				}
 			}
 		}
 		throw new ArgumentException("Value is not a dictionary type");
 	}
 }
 '@
`

