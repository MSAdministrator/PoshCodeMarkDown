---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 1.8
Encoding: ascii
License: cc0
PoshCode ID: 2459
Published Date: 2011-01-17t05
Archived Date: 2016-03-18t23
---

# convertfrom-fahrenheitwi - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 param([double] $Fahrenheit)
 
 Set-StrictMode -Version Latest
 
 function ConvertFahrenheitToCelsius([double] $fahrenheit)
 {
     $celsius = $fahrenheit - 32
     $celsius = $celsius / 1.8
     $celsius
 }
 
 $celsius = ConvertFahrenheitToCelsius $fahrenheit
 
 "$fahrenheit degrees Fahrenheit is $celsius degrees Celsius."
`

