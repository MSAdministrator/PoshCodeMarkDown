---
Author: dormidont
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6121
Published Date: 2016-11-27t21
Archived Date: 2016-03-18t22
---

# hex string to sid string - 

## Description

the function hex2sid will convert hex-represented sid string like 010500000000000515000000358a021a75b9755407e53b2b1ef10108 to sid string (also known as ssdl)

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 Function EndianReverse ($strHex)
 {
  $intCounter=$strHex.length-1
  do
   { 
    $reverse=$reverse+$strHex.substring($intCounter-1, 2)
    $intCounter=$intCounter-2
   }
  until ($intCounter -eq -1)
  return $reverse
 }
 
 Function hex2sid ($strHex)
 {
  $intSidVersionLength = 2
  $intSubAuthorityCountLength = 2
  $intAuthorityIdentifierLength = 12
  $intSubAuthorityLength = 8
  $intStringPosition = 0
  $bytSidVersion = [byte][convert]::ToInt32($strHex.substring($intStringPosition, $intSidVersionLength),16)
  $intStringPosition = $intStringPosition + $intSidVersionLength
  $bytSubAuthorityCount=[byte][convert]::ToInt32($strHex.substring($intStringPosition, $intSubAuthorityCountLength),16)
  $intStringPosition = $intStringPosition + $intSubAuthorityCountLength
  $lngAuthorityIdentifier=[long][convert]::ToInt32($strHex.substring($intStringPosition, $intAuthorityIdentifierLength),16)
  $intStringPosition = $intStringPosition + $intAuthorityIdentifierLength
  [string]$ConvertHexStringToSidString = "S-" + $bytSidVersion + "-" + $lngAuthorityIdentifier
  Do 
   {
    $lngTempSubAuthority = EndianReverse($strHex.substring($intStringPosition, $intSubAuthorityLength))
    $lngTempSubAuthority = [long][convert]::ToInt32($lngTempSubAuthority,16)
    $intStringPosition = $intStringPosition + $intSubAuthorityLength
    if ($lngTempSubAuthority -lt 0) 
     {
      $lngTempSubAuthority = $lngTempSubAuthority + 4294967296
     }
    $ConvertHexStringToSidString = $ConvertHexStringToSidString+"-"+$lngTempSubAuthority
    $bytSubAuthorityCount = $bytSubAuthorityCount - 1
   }
  until ($bytSubAuthorityCount -eq 0)
  return $ConvertHexStringToSidString
 }
`

