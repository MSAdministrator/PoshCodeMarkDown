---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6295
Published Date: 2016-04-08t21
Archived Date: 2016-04-11t15
---

# password generator - 

## Description

functions

## Comments



## Usage



## TODO



## function

`check-even`

## Code

`#
 #
 $PasswdList = Import-CSV $ENV:UserProfile\words.csv
 
 function Check-Even($num){[bool]!($num%2)}
 
 function New-PIN{ 
 	[CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Low')] 
 	param( 
 		[parameter(Mandatory = $true, Position = 0)] 
 		[ValidateRange(1,([int]::MaxValue))][int]$Digits = 1 
 	) 
 	$NC = 0
 	[string]$PIN = ""
 	While($NC -lt $Digits){
 		$PIN += Get-Random -Minimum 0 -Maximum 10
 		$NC += 1
 	} 
 	If($Digits -lt 19){
 		return [int64]$PIN}
 	Else{return [string]$PIN}
 }
 
 function New-Password{ 
     [CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Low')] 
     param( 
         [parameter(Mandatory = $true, Position = 0)] 
         [ValidateRange(1,([int]::MaxValue))][int]$Total = 12,
         [ValidateRange(0,([int]::MaxValue))][int]$Upper = 0,
         [ValidateRange(0,([int]::MaxValue))][int]$Lower = 0,
         [ValidateRange(0,([int]::MaxValue))][int]$Numeric = 0,
         [ValidateRange(0,([int]::MaxValue))][int]$Symbol = 0,
     ) 
     Process{ 
     if(($Upper+$Lower+$Numeric) -gt $Total){ 
         throw "New-Password : Cannot validate argument on parameter 'Total'. The $Total argument is less than the sum of parameters 'Upper', 'Lower', 'Numeric', and 'Symbol'. 
     Supply an argument that is greater than or equal to the sum of parameters 'Upper', 'Lower', 'Numeric', and 'Symbol'." 
     } 
 	$UC = (65..90)
 	$LC = (97..122)
 	$NU = (48..57)
 	$SY = (33..38 + 40..47 + 58..64 + 91..95 + 123..126)
     [int[]]$IArr = New-Object System.Int32 
     If($Upper -ge 1){ 
         $IArr += Get-Random -Input $($UC) -Count $Upper 
     } 
     If($Lower -ge 1){ 
         $IArr += Get-Random -Input $($LC) -Count $Lower 
     } 
     If($Numeric -ge 1){ 
         $IArr += Get-Random -Input $($NU) -Count $Numeric 
     } 
     If($Symbol -ge 1){ 
         $IArr += Get-Random -Input $($SY) -Count $Symbol 
     } 
     If($Total -gt ($Upper+$Lower+$Numeric+$Symbol)){ 
         $IArr += Get-Random -Input $($UC + $LC + $NU + $SY) -Count ($Total - $Upper - $Lower - $Numeric - $Symbol) 
     } 
     $IArr =  $IArr -ne 0 
     return ([char[]](Get-Random -InputObject $IArr -Count $IArr.Count)) -join "" 
     } 
 }
 
 function New-PassPhrase{ 
 [CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Low')] 
 param( 
 	[parameter(Mandatory = $false, Position = 0)] 
 	[ValidateRange(1,([int]::MaxValue))][int]$Words = 4,
 	[parameter(Mandatory = $false, Position = 1)] 
 	[ValidateRange(0,([int]::MaxValue))][int]$Numbers = 4,
 	[ValidateSet("~","!","@","#","$","%","^","&","*","-","_","=","+",";",":",",",".","|"," ")][string]$Separator = "-"
 ) 
 
 	$WC = 0
 	While($WC -lt $Words){
 	$Case = (Get-Random -Minimum 0 -Maximum 100)
 	If((Check-Even $Case) -eq $True){ 
 		$Word += ($(Get-Random -Input $PasswdList).Word).ToUpper()
 		$Word += $Separator
 		$WC += 1
 	} 
 	elseIf((Check-Even $Case) -eq $False){
 		$Word += ($(Get-Random -Input $PasswdList).Word).ToLower()
 		$Word += $Separator
 		$WC += 1
 	}
 	}
 
 	$NC = 0
 	If($Words -eq 0){[string]$Word = ""}
 	While($NC -lt $Numbers){
 		$Word += Get-Random -Minimum 0 -Maximum 10
 		$NC += 1
 	} 
 	return [string]$Word
 }
`

