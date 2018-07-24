---
Author: michal
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6345
Published Date: 2016-05-16t09
Archived Date: 2016-10-05t15
---

# 2qyhb-ckdxc-t6t6x-w686f- - 

## Description

retrieve the windows product key of a specified machine using wmi

## Comments



## Usage



## TODO



## function

`get-windowsproductkey`

## Code

`#
 #
 function get-windowsproductkey([string]$computer)
 {
 $Reg = [WMIClass] ("\\" + $computer + "\root\default:StdRegProv")
 $values = [byte[]]($reg.getbinaryvalue(2147483650,"SOFTWARE\Microsoft\Windows NT\CurrentVersion","DigitalProductId").uvalue)
 $lookup = [char[]]("B","C","D","F","G","H","J","K","M","P","Q","R","T","V","W","X","Y","2","3","4","6","7","8","9")
 $keyStartIndex = [int]52;
 $keyEndIndex = [int]($keyStartIndex + 15);
 $decodeLength = [int]29
 $decodeStringLength = [int]15
 $decodedChars = new-object char[] $decodeLength 
 $hexPid = new-object System.Collections.ArrayList
 for ($i = $keyStartIndex; $i -le $keyEndIndex; $i++){ [void]$hexPid.Add($values[$i]) }
 for ( $i = $decodeLength - 1; $i -ge 0; $i--)
     {                
      if (($i + 1) % 6 -eq 0){$decodedChars[$i] = '-'}
      else
        {
         $digitMapIndex = [int]0
         for ($j = $decodeStringLength - 1; $j -ge 0; $j--)
         {
             $byteValue = [int](($digitMapIndex * [int]256) -bor [byte]$hexPid[$j]);
             $hexPid[$j] = [byte] ([math]::Floor($byteValue / 24));
             $digitMapIndex = $byteValue % 24;
             $decodedChars[$i] = $lookup[$digitMapIndex];
          }
         }
      }
 $STR = ''     
 $decodedChars | % { $str+=$_}
 $STR
 }
 get-windowsproductkey .
`

