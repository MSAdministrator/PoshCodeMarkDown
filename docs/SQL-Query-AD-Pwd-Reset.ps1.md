---
Author: vidrine
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3336
Published Date: 2014-04-10t11
Archived Date: 2014-08-18t21
---

# sql query - ad pwd reset - 

## Description

script connects to a sql database and runs a query against the specified table. depending on table record values, an active directory user object will have it’s password reset.  once, the account is reset the sql record is updated.

## Comments



## Usage



## TODO



## script

`get-timestamp`

## Code

`#
 #
 <#
 .SYNOPSIS
   Author:......Vidrine
   Date:........2012/04/08
 .DESCRIPTION
   Script connects to a SQL database and runs a query against the specified table. Depending on table record values, 
   an Active Directory user object will have it's password reset.  Once, the account is reset the SQL record is updated.
   This SQL update is to prevent resetting the user object's password, again, and to store the password for use.
 .NOTES
   Requirements:
   .. Microsoft ActiveDirectory cmdlets
   .. Microsoft SQL cmdlets
   
   Additionally:
   The script must be ran as account that has access to the database and access to 'reset passwords' within ActiveDirectory.
 #>
 
 ##====================================================================================
 ##====================================================================================
 add-pssnapin SqlServerCmdletSnapin100 -ErrorAction SilentlyContinue
 add-pssnapin SqlServerProviderSnapin100 -ErrorAction SilentlyContinue
 Import-Module activeDirectory -ErrorAction SilentlyContinue
 
 ##====================================================================================
 ##====================================================================================
 $sqlDatabase        = 'DatabaseName'
 $sqlTable           = 'TableName'
 
 ##====================================================================================
 ##====================================================================================
 $word    = Get-Content "C:\..\5CharacterDictionary.txt"
 $special ='!','@','#','$','%','^','&','*','(',')','-','_','+','='
 $nmbr    = 4
 
 ##====================================================================================
 ##====================================================================================
 $logFile = (Get-Date -Format yyyyMMdd) + '_LogFile.csv'
 $logPath = 'C:\..\Logs'
 $log     = Join-Path -ChildPath $logFile -Path $logPath
 
 ##====================================================================================
 ##====================================================================================
 function Get-Timestamp {
 	Get-Date -Format u
 }
 
 function Write-Log {
 	param(
 		[string] $Path,
 		[string] $Value
 	)
 
 	$Value | Out-File $Path -Append
 }
 
 function Create-Password {
 	$NewString = ""
 	1..$nmbr | ForEach { $NewString = $NewString + (Get-Random -Minimum 0 -Maximum 9) }
 
 	$lowerWord		= Get-Random $word
 
 	$firstLetters	= $lowerWord.Substring(0,2)
 	$upperLetters	= $lowerWord.Substring(2,1).toUpper()
 	$lastLetters	= $lowerWord.Substring(3,2)
 	$NewWord		= $firstLetters + $upperLetters + $lastLetters
 
 	$NewSpecial = Get-Random $special
 	
 	$NewPassword = ($NewWord + $NewSpecial + $NewString)
 	
 	return $NewPassword
 }
 
 Function Reset-Password {
 	param (
 		[string]$emailAddress,
 		[string]$password
 	)
 
 	$password_secure = ConvertTo-SecureString $password -AsPlainText -Force
 		
 	try
 	{	
 		Get-ADUser -Filter {emailAddress -like $emailAddress} | Set-ADAccountPassword -Reset -NewPassword $password_secure
 		$Value = (get-timestamp)+"`tSUCCESS`tReset Password`tPassword reset completed for end user ($emailAddress)."
 		Write-Log -Path $log -Value $Value
 	}
 	catch
 	{
 		$Value = (get-timestamp)+"`tERROR`tReset Password`tUnable to reset password ($emailAddress). $_"
 		Write-Log -Path $log -Value $Value
 	}
 }
 
 function Get-Username {
 	param (
 		[string]$emailAddress
 	)
 	
 	try
 	{
 		$user = Get-ADUser -Filter {emailAddress -like $emailAddress}
 		$Value = (get-timestamp)+"`tSUCCESS`tQuery Username`tDirectory lookup for username was successful ($emailAddress)."
 		Write-Log -Path $log -Value $Value
 		
 		return $user.sAMAccountName
 	}
 	catch
 	{
 		$Value = (get-timestamp)+"`tERROR`tQuery Username`tDirectory lookup failed ($emailAddress). $_"
 		Write-Log -Path $log -Value $Value
 	}
 }
 
 function SQL-Select {
 <#
 .EXAMPLE
 $results = SQL-Select -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -selectWhat '*'
 .EXAMPLE
 $results = SQL-Select -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -selectWhat '*' -where "id='64'"
 #>
 
 param(
 	[string]$server,
 	[string]$database,
 	[string]$table,
 	[string]$selectWhat,
 	[string]$where
 	
 )
 
 if ($where){
 $sqlQuery = @"
 SELECT $selectWhat 
 FROM $table 
 WHERE $where
 "@
 }
 
 else {
 $sqlQuery = @"
 SELECT $selectWhat 
 FROM $table
 "@
 }
 
 try
 {
 $results = Invoke-SQLcmd -ServerInstance $server -Database $database -Query $sqlQuery
 $Value = (get-timestamp)+"`tSUCCESS`tSQL Select`tDatabase query was successful (WHERE: $where)."
 Write-Log -Path $log -Value $Value
 
 return $results
 }
 catch
 {
 $Value = (get-timestamp)+"`tERROR`tSQL Select`tDatabase query failed (WHERE: $where). $_"
 Write-Log -Path $log -Value $Value
 }
 }
 
 function SQL-Update {
 <#
 .EXAMPLE
 SQL-Update -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -dataField $sqlDataField -dataValue $sqlDataValue -updateID $sqlDataID
 #>
 param(
 	[string]$server,
 	[string]$database,
 	[string]$table,
 	[string]$dataField,
 	[string]$dataValue,
 	[string]$updateID
 )
 
 $sqlQuery = @"
 UPDATE $database.$table 
 SET $dataField='$dataValue' 
 WHERE id=$updateID
 "@
 
 try
 {
 Invoke-SQLcmd -ServerInstance $server -Database $database -Query $sqlQuery
 $Value = (get-timestamp)+"`tSUCCESS`tSQL Update`tUpdated database record, ID $updateID ($dataField > $dataValue)."
 Write-Log -Path $log -Value $Value
 }
 catch
 {
 $Value = (get-timestamp)+"`tERROR`tSQL Update`tUnable to update database record, ID $updateID ($dataField > $dataValue). $_"
 Write-Log -Path $log -Value $Value
 }
 }
 
 function Check-Status {
 	$results = $NULL
 	$results = SQL-Select -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -selectWhat 'id,email,pword,pwordSet,status' -where "(pwordSet IS Null OR pwordSet='') AND status='CheckedIn'"
 	$results | ForEach {
 		if ($_.pword.GetType().name -eq 'DBNull')
 		{
 			$password = Create-Password
 			
 			$sqlDataID = $_.id
 			
 			$sqlDataField = 'pword'
 			$sqlDataValue = $password
 			SQL-Update -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -dataField $sqlDataField -dataValue $sqlDataValue -updateID $sqlDataID
 			
 			Reset-Password -emailAddress $_.email -password $password
 			
 			$sqlDataField = 'pwordSet'
 			$sqlDataValue = 'Yes'
 			SQL-Update -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -dataField $sqlDataField -dataValue $sqlDataValue -updateID $sqlDataID
 			
 			$sqlDataField = 'samaccountname'
 			$sqlDataValue = Get-Username -emailAddress $_.email
 			SQL-Update -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -dataField $sqlDataField -dataValue $sqlDataValue -updateID $sqlDataID
 		}
 		elseif($_.pword -eq '')
 		{
 			$password = Create-Password
 			
 			$sqlDataID = $_.id
 			
 			$sqlDataField = 'pword'
 			$sqlDataValue = $password
 			SQL-Update -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -dataField $sqlDataField -dataValue $sqlDataValue -updateID $sqlDataID
 			
 			Reset-Password -emailAddress $_.email -password $password
 			
 			$sqlDataField = 'pwordSet'
 			$sqlDataValue = 'Yes'
 			SQL-Update -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -dataField $sqlDataField -dataValue $sqlDataValue -updateID $sqlDataID
 			
 			$sqlDataField = 'samaccountname'
 			$sqlDataValue = Get-Username -emailAddress $_.email
 			SQL-Update -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -dataField $sqlDataField -dataValue $sqlDataValue -updateID $sqlDataID
 		}
 		else 
 		{
 			Reset-Password -emailAddress $_.email -password $_.pword
 			
 			$sqlDataID    = $_.id
 			
 			$sqlDataField = 'pwordSet'
 			$sqlDataValue = 'Yes'
 			SQL-Update -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -dataField $sqlDataField -dataValue $sqlDataValue -updateID $sqlDataID
 			
 			$sqlDataField = 'samaccountname'
 			$sqlDataValue = Get-Username -emailAddress $_.email
 			SQL-Update -server $sqlServerInstance -database $sqlDatabase -table $sqlTable -dataField $sqlDataField -dataValue $sqlDataValue -updateID $sqlDataID
 		}
 	}
 	return $results
 }
 
 ##====================================================================================
 ##====================================================================================
 Check-Status
`

