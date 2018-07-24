---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2721
Published Date: 2011-06-09t13
Archived Date: 2011-12-15t09
---

# select-clscompliant - 

## Description

the beginnings of a function for handling ets exceptions thrown by types which are not cls compliant when you try to output them.

## Comments



## Usage



## TODO



## function

`select-clscompliant`

## Code

`#
 #
 function Select-CLSCompliant {
 #.Synopsis
 
 [CmdletBinding()]
 param([Parameter(ValueFromPipeline=$true)]$InputObject)
 process {
    foreach($in in $InputObject) {
       $In | Select-Object *
 
       trap [System.Management.Automation.ExtendedTypeSystemException] {
          $m = $_.Exception.Message
          $matches = [regex]::Matches($m, 'The field/property: \"(?<field>.*)\" for type: \"(?<Type>[^"]+)\" .* Failed to use non CLS compliant type.')
          $type = $matches[0].Groups["Type"].Value -as [Type]
          
             
          $properties = $type.GetProperties()
          $output = @{}
          $properties | %{ $Output.($_.Name) = $_.GetValue( $In, $null ) }
          $NewObject = new-Object PSObject -Property $Output
          $NewObject.pstypenames.insert(0,"Selected." + ($matches[0].Groups["Type"].Value))
          Write-Output $NewObject
          continue
       }
    }
 }}
`

