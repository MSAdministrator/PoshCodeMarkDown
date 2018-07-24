---
Author: vince ypma
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5660
Published Date: 2016-01-03t01
Archived Date: 2016-05-19t06
---

# convertto-romannumeral - 

## Description

this is submitted as a companion to the convertfrom-romannumeral function just

## Comments



## Usage



## TODO



## function

`convertto-romannumeral`

## Code

`#
 #
 function ConvertTo-RomanNumeral
 {
   <#
     .SYNOPSIS
         Converts a number to a Roman numeral.
     .DESCRIPTION
         Converts a number - in the range of 1 to 3,999 - to a Roman numeral.
     .PARAMETER Number
         An integer in the range 1 to 3,999.
     .INPUTS
         System.Int32
     .OUTPUTS
         System.String
     .EXAMPLE
         ConvertTo-RomanNumeral -Number (Get-Date).Year
     .EXAMPLE
         (Get-Date).Year | ConvertTo-RomanNumeral
   #>
     [CmdletBinding()]
     [OutputType([string])]
     Param
     (
         [Parameter(Mandatory=$true,
                    HelpMessage="Enter an integer in the range 1 to 3,999",
                    ValueFromPipeline=$true,
                    Position=0)]
         [ValidateRange(1,3999)]
         [int]
         $Number
     )
 
     Begin
     {
         $DecimalToRoman = @{
             Thousands = "","M","MM","MMM"
             Hundreds  = "","C","CC","CCC","CD","D","DC","DCC","DCCC","CM"
             Tens      = "","X","XX","XXX","XL","L","LX","LXX","LXXX","XC"
             Ones      = "","I","II","III","IV","V","VI","VII","VIII","IX"
         }
     
         $column = @{
             Thousands = 0
             Hundreds  = 1
             Tens      = 2
             Ones      = 3
         }
     }
     Process
     {
         [int[]]$digits = $Number.ToString().PadLeft(4,"0").ToCharArray() |
                             ForEach-Object { [Char]::GetNumericValue($_) }
 
         $RomanNumeral  = ""
         $RomanNumeral += $DecimalToRoman.Thousands[$digits[$column.Thousands]]
         $RomanNumeral += $DecimalToRoman.Hundreds[$digits[$column.Hundreds]]
         $RomanNumeral += $DecimalToRoman.Tens[$digits[$column.Tens]]
         $RomanNumeral += $DecimalToRoman.Ones[$digits[$column.Ones]]
 
         $RomanNumeral
     }
     End
     {
     }
 }
`

