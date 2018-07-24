---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5190
Published Date: 2014-05-24t03
Archived Date: 2014-05-26t10
---

# scalar context - 

## Description

adds a scalar variables to the powershell type system.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 <#
 
 The problem with PowerShell is that it uses .NET's type system instead of its own.
 
 In .NET, [object] is the top of the type hierarchy, [object[]] is below it and there is no common base class that all scalar types derive from.
 
 This script is a hack that makes the PowerShell type system operate more like this:
 
                     [object]
            /                        \
      [scalar()]             any unravelable type with > 1 elements
    /     |     \            /          |        \
 [int] [string] ...       [object[]] [arraylist] ...
 
 
 $null and [AutomationNull]::Value are considered scalars
 unravelable types with 0 or 1 elements are considered scalars and will be automatically unravelled
 
 Basically:
 
 [scalar()]$x = $rhs
 
 is like:
 
 $x, $null = $rhs
 
 Except:
 1. it fails if $rhs would unravel to more than 1 element rather than silently continuing
 2. it preserves AutomationNull correctly unlike PowerShell
 
 NOTE:
 This only unravels 1 level deep.
 
 #>
 
 Add-Type -TypeDefinition @'
 using System;
 using System.Management.Automation;
 
 [AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
 public sealed class ScalarAttribute : ArgumentTransformationAttribute {
 	public override Object Transform(EngineIntrinsics engineIntrinsics, Object inputData) {
 		System.Collections.IEnumerator enumerator = LanguagePrimitives.GetEnumerator(inputData);
 		if (enumerator != null) {
 			inputData = System.Management.Automation.Internal.AutomationNull.Value;
 			for (int i = 0; enumerator.MoveNext();) {
 				if (++i == 2)
 					throw new ArgumentException("Sequence contains more than 1 element");
 				inputData = enumerator.Current;
 			}
 		}
 		return inputData;
 		
 	}
 }
 '@
`

