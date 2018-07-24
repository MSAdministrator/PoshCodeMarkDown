---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2094
Published Date: 2012-08-19t11
Archived Date: 2012-01-08t21
---

# themathfunction - 

## Description

you need to download and unpack loresoft.mathexpressions.dll into your documents\windowspowershell\libraries or tweak this module.

## Comments



## Usage



## TODO



## function

`use-math`

## Code

`#
 #
 Add-Type -Path (Join-Path (Split-Path $Profile) Libraries\LoreSoft.MathExpressions.dll)
 
 $MathEvaluator = New-Object LoreSoft.MathExpressions.MathEvaluator
 
 Function Use-Math {
    $MathEvaluator.Evaluate( ($args -join " ") )
 }
 
 Set-Alias Math Use-Math
 
 Export-ModuleMember -Function * -Alias *
`

