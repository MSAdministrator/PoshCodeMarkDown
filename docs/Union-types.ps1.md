---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5314
Published Date: 2016-07-17t03
Archived Date: 2016-05-19t10
---

# union types - 

## Description

this module is a hack to add union types to powershell.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 
 After running this code, use it like:
 
 [or([int],[float])]$lhs = rhs
 
 ##########
 ##########
 
 ####
 1. If rhs is $null, it will examine all of the types in the or() to determine if they can accept $null.
 If all the types are value types, $null wont be accepted as a valid value. If you want to allow nulls in this scenario, use [Nullable[some_valuetype]] or [Nullable`1] as at least one of the types.
 
 
 ####
 2. or() uses $Type.IsAssignableFrom which checks the inheritence hierarchy (i.e. it's similar to -is) rather than the fancier PowerShell algorithm which considers casts, constructors, conversion through strings or a numeric tower, etc. e.g.
 
 [or([long])]$a = 1
 
 [or([string])]$a = $null
 
 [or([int])]$a = $null
 
 In other words, [or([foo])] works similar to [ValidateScript({$_ -is [foo]})]
 
 
 ####
 3. or() allows generic type definitions unlike powershell, e.g.
 
 [or([System.Collections.Generic.ISet`1])]$set = [System.Collections.Generic.HashSet[int]]@(1, 2, 3)
 
 However, it only works on the top level type. e.g.
 
 [or([dictionary[int,dictionary`2]])]
 
 
 ####
 4. [or()] without a types listed will always fail when you try to assign to it.
 
 
 #>
 
 
 Add-Type -TypeDefinition @'
 using System;
 using System.Management.Automation;
 
 [AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
 public class OrAttribute : ArgumentTransformationAttribute {
 	private Type[] _types;
 
 	public OrAttribute(params Type[] types) {
 		this._types = types ?? System.Type.EmptyTypes;
 	}
 
 	private static bool matches(Type wanted, Type inputDataType) {
 		if (wanted.IsInterface) {
 			foreach (Type ti in inputDataType.GetInterfaces()) {
 				if (ti.IsGenericType && wanted == ti.GetGenericTypeDefinition())
 					return true;
 			}
 		} else {
 			do {
 				if (inputDataType.IsGenericType && wanted == inputDataType.GetGenericTypeDefinition())
 					return true;
 			} while ((inputDataType = inputDataType.BaseType) != null);
 		}
 		return false;
 	}
 
 	public override Object Transform(EngineIntrinsics engineIntrinsics, Object inputData) {
 		Object o = inputData;
 		PSObject pso = inputData as PSObject;
 		if (pso != null)
 			o = (pso != System.Management.Automation.Internal.AutomationNull.Value) ? pso.BaseObject : null;
 
 		if (o == null) {
 			foreach (Type t in this._types) {
 				if (!t.IsValueType || (t.IsGenericType && t.GetGenericTypeDefinition() == typeof(Nullable<>)))
 					return inputData;
 			}
 		} else {
 			Type inputDataType = inputData.GetType();
 			foreach (Type t in this._types) {
 				if ((t.IsGenericTypeDefinition && matches(t, inputDataType)) || t.IsAssignableFrom(inputDataType))
 					return inputData;
 			}
 		}
 		throw new ArgumentException("Value doesn't match any union types");
 	}
 }
 '@
`

