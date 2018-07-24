---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5473
Published Date: 2014-10-01t06
Archived Date: 2014-10-10t23
---

# set-localuserpwd - 

## Description

adaptation of script to set local admin password thru sccm

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 [CmdletBinding()]
 param(
 $computername = $env:COMPUTERNAME,
 #$username = "Administrator",
 $username = (gwmi Win32_UserAccount -Filter "LocalAccount = True AND SID LIKE 'S-1-5-21-%-500'" -ComputerName $computerName | Select -First 1 ).Name,
 $password,
 [switch] $Test
 )
 
 process{
 $VerbosePreference = "Continue"
 if (-not $password){$password = GeneratePassword}
 if (-not $test){
 	try{
 		([adsi]("WinNT://$($computerName)/$($username)")).SetPassword($password)
 		SaveToSCCMDB $computername $password
 		Write-Verbose "Password $password set for $username on $computername"
 		}
 	catch{
 		Write-Verbose "Error while setting password $password for $username on $computername"
 		}
 	}
 else{
 	Write-Verbose "TEST: Password $password set for $username on $computername"
 	write-host "The generated password is $password"
 	}
 }
 
 
 begin{
 $script:verbosepref = $VerbosePreference
 function GeneratePassword {
 param(
 [byte]$LowerCase = 4, 
 [byte]$UpperCase = 2, 
 [byte]$Numbers = 2, 
 [byte]$Specials = 0, 
 [switch]$AvoidAmbiguous = $true
 )
 
 if ($AvoidAmbiguous){
 	$arrLCase = "abcdefghijkmnpqrstuvwxyz".ToCharArray()
 	$arrUCase = "ABCDEFGHJKLMNPQRSTUVWXYZ".ToCharArray()
 	$arrNum = "23456789".ToCharArray()
 	$arrSpec = "!#$%&()*+,-./:;<=>?@[\]^_{}~".ToCharArray()
 	}
 else{
 	$arrLCase = "abcdefghijklmnopqrstuvwxyz".ToCharArray()
 	$arrUCase = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".ToCharArray()
 	$arrNum = "1234567890".ToCharArray()
 	$arrSpec = "!#$%&()*+,-./:;<=>?@[\]^_{}~|".ToCharArray()
 	}
 $aCharacters = @()
 	
 if ($LowerCase -gt 0){$aCharacters += $arrLCase | get-random -count $LowerCase}
 if ($UpperCase -gt 0){$aCharacters += $arrUCase | get-random -count $UpperCase}
 if ($Numbers -gt 0){$aCharacters += $arrNum | get-random -count $Numbers}
 if ($Specials -gt 0){$aCharacters += $arrSpec | get-random -count $Specials}
 $result = [string]::join("", $($aCharacters | get-random -count $aCharacters.length))
 Write-Verbose "generated password = $result"
 return $result
 
 function SaveToSCCMDB ($computername,$password){
 try{
 	$conn = New-Object System.Data.SqlClient.SqlConnection("Server=sccmsql.domain.com; Database=LocalPwd; User Id=lpwd;Password=hiddenpwd;")
 	$conn.Open()
 	$cmd = $conn.CreateCommand()
 	$cmd.CommandText = "EXEC [dbo].[SetLocalAdminPassword] @ComputerName = N'$computername', @Password = N'$password'"
 	$cmd.ExecuteNonQuery()
 	$conn.Close()
 	}
 catch{}
 }
 
 end{
 $VerbosePreference = $script:verbosepref
 }
`

