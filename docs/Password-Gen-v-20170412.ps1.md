---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6841
Published Date: 2017-04-12t22
Archived Date: 2017-05-30t04
---

# password gen v.20170412 - 

## Description

functions

## Comments



## Usage



## TODO



## function

`compare-even`

## Code

`#
 #
 	[bool]!($num%2)
 }
 
 Function New-PIN{ 
 #
 #
 #
 #
 #
 #
 #
 #
 #
 	[CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Low')] 
 	param( 
 		[parameter(Mandatory = $false, Position = 0)] 
 		[ValidateRange(1,([int]::MaxValue))][int]$Digits = 4,
 		[parameter(Mandatory = $false, Position = 1)]
 		[ValidateRange(1,([int]::MaxValue))][int]$Generate = 1
 	) 
 	For($i=0;$i -lt $Generate;$i++){
 		$NC = 0
 		[string]$PIN = ""
 		While($NC -lt $Digits){
 			$PIN += Get-Random -Minimum 0 -Maximum 10
 			$NC += 1
 		} 
 		Write-Host $PIN.Length "digits:  " -NoNewLine
 		Write-Host $PIN
 		Remove-Variable PIN
 	}
 }
 
 Function New-Password{ 
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 
 	[CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Low')] 
 	param( 
 		[parameter(Mandatory = $false, Position = 0)] 
 		[ValidateRange(1,([int]::MaxValue))][int]$Total = 16,
 		[ValidateSet("All","AlphaNumericMixed","AlphaNumericUpper","AlphaNumericLower","AlphaSymbolMixed","AlphaSymbolUpper","AlphaSymbolLower","AlphaMixed","AlphaLower","AlphaUpper","SymbolNumeric","SymbolOnly")][string]$CharacterSet = "All",
 		[parameter(Mandatory = $false, Position = 1)]
 		[ValidateRange(1,([int]::MaxValue))][int]$Generate = 1
 	) 
 	Process{ 
 		For($i=0;$i -lt $Generate;$i++){
 			[string]$Passwd = ''
 			$UC = $NULL;For($a=65;$a -le 90;$a++){$UC+=,[char][byte]$a}
 			$LC = $NULL;For($a=97;$a -le 122;$a++){$LC+=,[char][byte]$a}
 			$NU = $NULL;For($a=48;$a -le 57;$a++){$NU+=,[char][byte]$a}
 			$SY = ("!","@","#","$","%","^","&","*","(",")","{","}","[","]","\","|","/","?","<",">",",",".","-","+","_","=",";",":")
 			If($CharacterSet -eq "All"){
 				$BaseSet = $UC + $LC + $NU + $SY}
 			elseIf($CharacterSet -eq "AlphaNumericMixed"){
 				$BaseSet = $UC + $LC + $NU}
 			elseIf($CharacterSet -eq "AlphaNumericUpper"){
 				$BaseSet = $UC + $NU}
 			elseIf($CharacterSet -eq "AlphaNumericLower"){
 				$BaseSet = $LC + $NU}
 			elseIf($CharacterSet -eq "AlphaSymbolMixed"){
 				$BaseSet = $UC + $LC + $SY}
 			elseIf($CharacterSet -eq "AlphaSymbolUpper"){
 				$BaseSet = $UC + $SY}
 			elseIf($CharacterSet -eq "AlphaSymbolLower"){
 				$BaseSet = $LC + $SY}
 			elseIf($CharacterSet -eq "AlphaMixed"){
 				$BaseSet = $LC + $UC}
 			elseIf($CharacterSet -eq "AlphaLower"){
 				$BaseSet = $LC}
 			elseIf($CharacterSet -eq "AlphaUpper"){
 				$BaseSet = $UC}
 			elseIf($CharacterSet -eq "SymbolNumeric"){
 				$BaseSet = $SY + $NU}
 			elseIf($CharacterSet -eq "SymbolOnly"){
 				$BaseSet = $SY}
 			For ($Loop=1;$Loop -le $Total;$Loop++) {
 				$Passwd += Get-Random -Input $($BaseSet)
 			} 
 			Write-Host $Passwd.Length "char:  " -NoNewLine
 			Write-Host $Passwd
 			Remove-Variable Passwd
 		}
 	}
 }
 
 Function New-PassPhrase{ 
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 
 [CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Low')] 
 param( 
 	[parameter(Mandatory = $false, Position = 0)] 
 	[ValidateRange(1,([int]::MaxValue))][int]$Words = 4,
 	[parameter(Mandatory = $false, Position = 1)]
 	[ValidateRange(1,([int]::MaxValue))][int]$WordLength = 0,
 	[parameter(Mandatory = $false, Position = 2)] 
 	[ValidateRange(0,([int]::MaxValue))][int]$NumbersBefore = 0,
 	[parameter(Mandatory = $false, Position = 3)] 
 	[ValidateRange(0,([int]::MaxValue))][int]$NumbersAfter = 4,
 	[parameter(Mandatory = $false, Position = 4)] 
 	[ValidateRange(0,([int]::MaxValue))][int]$SymbolsBefore = 0,
 	[parameter(Mandatory = $false, Position = 5)] 
 	[ValidateRange(0,([int]::MaxValue))][int]$SymbolsAfter = 0,
 	[parameter(Mandatory = $false, Position = 6)] 
 	[ValidateRange(0,([int]::MaxValue))][int]$PadTo = 0,
 	[parameter(Mandatory = $false, Position = 7)]
 	[ValidateSet("Alternating","InvertTitleCase","LowerCase","Random","TitleCase","UpperCase")][string]$Case = "Alternating",
 	[parameter(Mandatory = $false, Position = 8)]
 	[ValidateSet("~","!","@","#","$","%","^","&","*","-","_","=","+",";",":",",",".","\","|","/"," ","(",")","[","]","{","}","<",">","?",$null)]
 	[string]$Separator = (Get-Random -Input ("~","!","@","#","$","%","^","&","*","-","_","=","+",";",":",",",".","|"," ")),
 	[parameter(Mandatory = $false, Position = 9)]
 	[ValidateSet("~","!","@","#","$","%","^","&","*","-","_","=","+",";",":",",",".","\","|","/"," ","(",")","[","]","{","}","<",">","?")]
 	[string]$PadSymbol = (Get-Random -Input ("~","!","@","#","$","%","^","&","*","-","_","=","+",";",":",",",".","|")),
 	[parameter(Mandatory = $false, Position = 10)]
 	[ValidateRange(1,([int]::MaxValue))][int]$Generate = 1,
 	[parameter(Mandatory = $false, Position = 11)]
 	[switch]$RandomLen = $false,
 	[parameter(Mandatory = $false, Position = 12)]
 	[ValidateRange(4,([int]::MaxValue))][int]$MinLen = 4,
 	[parameter(Mandatory = $false, Position = 13)]
 	[ValidateScript({
 		if($_ -lt $MinLen){
 			throw 'MaxLen value cannot be less than MinLen value.'
 		} $true
 	})][int]$MaxLen = 8
 ) 
 
 	If ($WordLength -ne 0){
 		$PList = (Import-CSV $PasswdList.Location | Where { $_.Word.Length -eq $WordLength})
 	}else{
 		$PList = (Import-CSV $PasswdList.Location)
 	}
 	
 	If ($RandomLen -eq $true){
 		$PList = (Import-CSV $PasswdList.Location | Where { $_.Word.Length -ge $MinLen -AND $_.Word.Length -le $MaxLen})
 	}
 
 
 	For($i=0;$i -lt $Generate;$i++){
 
 		$bSY = 0
 		While($bSY -lt $SymbolsBefore){
 			[string]$Word += $PadSymbol
 			$bSY += 1
 		}
 
 		$bNC = 0
 		While($bNC -lt $NumbersBefore){
 			[string]$Word += Get-Random -Minimum 0 -Maximum 10
 			If($bNC -eq ($NumbersBefore - 1) -AND $bNC -gt 0){
 				[string]$Word += $Separator
 			}
 			$bNC += 1
 		} 
 
 		$WC = 0
 		While($WC -lt $Words){
 
 		$WordLength =  Get-Random -Minimum $MinLen -Maximum ($MaxLen+1)
 		
 			If($Case -eq "Random"){
 				$Rand = (Get-Random -Minimum 0 -Maximum 100)
 		
 				If((Compare-Even $Rand) -eq $True){ 
 					[string]$Word += ($(Get-Random -Input $PList).Word).ToUpper()
 					$WC += 1
 				}elseIf((Compare-Even $Rand) -eq $False){
 					[string]$Word += ($(Get-Random -Input $PList).Word).ToLower()
 					$WC += 1
 				}
 				If($WC -ne $Words){
 					[string]$Word += $Separator}
 
 			}elseIf($Case -eq "Alternating"){
 
 				If((Compare-Even $WC) -eq $True){ 
 					[string]$Word += ($(Get-Random -Input $PList).Word).ToUpper()
 					$WC += 1
 				}elseIf((Compare-Even $WC) -eq $False){
 					[string]$Word += ($(Get-Random -Input $PList).Word).ToLower()
 					$WC += 1
 				}
 				If($WC -ne $Words){
 					[string]$Word += $Separator}
 				
 			}elseIf($Case -eq "LowerCase"){
 					[string]$Word += ($(Get-Random -Input $PList).Word).ToLower()
 					$WC += 1
 				If($WC -ne $Words){
 					[string]$Word += $Separator}
 				
 			}elseIf($Case -eq "UpperCase"){
 					[string]$Word += ($(Get-Random -Input $PList).Word).ToUpper()
 					$WC += 1
 				If($WC -ne $Words){
 					[string]$Word += $Separator}
 			}elseIf($Case -eq "TitleCase"){
 					[string]$Word += (Get-Culture).TextInfo.ToTitleCase($(Get-Random -Input $PList).Word)
 					$WC += 1
 				If($WC -ne $Words){
 					[string]$word += $Separator}
 			}elseIf($Case -eq "InvertTitleCase"){
 					[string]$Word += -join (((Get-Culture).TextInfo.ToTitleCase(($(Get-Random -Input $PList).Word))).ToCharArray() | %{[char]([int][char]$_ -bxor 0x20)})
 					$WC += 1
 				If($WC -ne $Words){
 					[string]$word += $Separator}
 			}
 		}
 
 		$eNC = 0
 		If($Words -eq 0){[string]$Word += ""}
 		While($eNC -lt $NumbersAfter){
 			If($eNC -eq 0){
 				[string]$Word += $Separator}
 			[string]$Word += Get-Random -Minimum 0 -Maximum 10
 			$eNC += 1
 		}
 
 		$eSY = 0
 		While($eSY -lt $SymbolsAfter){
 			[string]$Word += $PadSymbol
 			$eSY += 1
 		}
 		
 		$Pad = 0
 		$WL = 0
 		[int]$WL = $Word.length
 		If($WL -lt $PadTo){
 			$Pad = ($PadTo - $WL)
 		}
 		While($WL -lt $PadTo){
 			[string]$Word += $PadSymbol
 			[int]$WL = $Word.length
 		}
 
 		Write-Host $Word.Length "char:  " -NoNewLine
 		Write-Host $Word
 		Remove-Variable Word
 		
 		If($Generate -gt 1){
 			If(!$PSBoundParameters.ContainsKey('Separator')){
 				[string]$Separator = (Get-Random -Input ("~","!","@","#","$","%","^","&","*","-","_","=","+",";",":",",",".","|"," "))
 			}
 			If(!$PSBoundParameters.ContainsKey('PadSymbol')){
 				[string]$PadSymbol = (Get-Random -Input ("~","!","@","#","$","%","^","&","*","-","_","=","+",";",":",",",".","|"))
 			}
 		}
 	}
 }
`

