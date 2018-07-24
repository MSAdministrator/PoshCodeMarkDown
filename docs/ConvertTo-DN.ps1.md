---
Author: glenn sizemore glnsize
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6445
Published Date: 2016-07-13t03
Archived Date: 2016-09-16t11
---

# convertto-dn - 

## Description

a couple functions i use to convert dn to canonical, and canonical to dn. i find them handy for adhoc ad tasks… i saw someone ask about it on #powershell and figured i would share

## Comments



## Usage



## TODO



## function

`convertfrom-dn`

## Code

`#
 #
 #
 #
 
 
 function ConvertFrom-DN 
 {
 param([string]$DN=(Throw '$DN is required!'))
     foreach ( $item in ($DN.replace('\,','~').split(",")))
     {
         switch -regex ($item.TrimStart().Substring(0,3))
         {
             "CN=" {$cn += ,$item.replace("CN=","");$cn += '/';continue}
             "OU=" {$ou += ,$item.replace("OU=","");$ou += '/';continue}
             "DC=" {$DC += $item.replace("DC=","");$DC += '.';continue}
         }
     } 
     $canoincal = $dc.Substring(0,$dc.length - 1)
     if ($ou -ne $null) { for ($i = $ou.count;$i -ge 0;$i -- ){$canoincal += $ou[$i]} }
     for ($i = $cn.count-1;$i -ge 0;$i -- ){$canoincal += $cn[$i].ToString().replace('~',',')}
     return $canoincal
 }
 
 function ConvertFrom-Canonical 
 {
 param([string]$canoincal=(trow '$Canonical is required!'))
     $obj = $canoincal.Replace(',','\,').Split('/')
     [string]$DN = "CN=" + $obj[$obj.count - 1]
     for ($i = $obj.count - 2;$i -ge 1;$i--){$DN += ",OU=" + $obj[$i]}
     $obj[0].split(".") | ForEach-Object { $DN += ",DC=" + $_}
     return $dn
 }
`

