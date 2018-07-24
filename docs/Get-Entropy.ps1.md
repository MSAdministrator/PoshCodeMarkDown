---
Author: matthew graeber
Publisher: 
Copyright: 
Email: 
Version: 0.0
Encoding: ascii
License: cc0
PoshCode ID: 3996
Published Date: 2013-03-03t08
Archived Date: 2013-03-10t01
---

# get-entropy - 

## Description

calculate the entropy of a byte array.

## Comments



## Usage



## TODO



## function

`get-entropy`

## Code

`#
 #
 function Get-Entropy
 {
 <#
 .SYNOPSIS
 
     Calculate the entropy of a byte array.
 
     Author: Matthew Graeber (@mattifestation)
 
 .PARAMETER ByteArray
 
     Specifies the byte array containing the data from which entropy will be calculated.
 
 .EXAMPLE
 
     C:\PS> $RandArray = New-Object Byte[](10000)
     C:\PS> foreach ($Offset in 0..9999) { $RandArray[$Offset] = [Byte] (Get-Random -Min 0 -Max 256) }
     C:\PS> $RandArray | Get-Entropy
 
     Description
     -----------
     Calculates the entropy of a large array containing random bytes.
 
 .EXAMPLE
 
     C:\PS> 0..255 | Get-Entropy
 
     Description
     -----------
     Calculates the entropy of 0-255. This should equal exactly 8.
 
 .INPUTS
 
     System.Byte[]
 
     Get-Entropy accepts a byte array from the pipeline
 
 .OUTPUTS
 
     System.Double
 
     Get-Entropy outputs a double representing the entropy of the byte array.
 
 .LINK
 
     http://www.exploit-monday.com
 #>
 
     [CmdletBinding()] Param (
         [Parameter(Mandatory = $True, Position = 0, ValueFromPipeline = $True)]
         [Byte[]]
         $ByteArray
     )
 
     BEGIN
     {
         $FrequencyTable = @{}
         $ByteArrayLength = 0
     }
 
     PROCESS
     {
         foreach ($Byte in $ByteArray)
         {
             $FrequencyTable[$Byte]++
             $ByteArrayLength++
         }
     }
 
     END
     {
         $Entropy = 0.0
 
         foreach ($Byte in 0..255)
         {
             $ByteProbability = ([Double] $FrequencyTable[[Byte]$Byte]) / $ByteArrayLength
             if ($ByteProbability -gt 0)
             {
                 $Entropy += -$ByteProbability * [Math]::Log($ByteProbability, 2)
             }
         }
 
         Write-Output $Entropy
     }
 }
`

