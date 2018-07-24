---
Author: matt wilson
Publisher: 
Copyright: 
Email: 
Version: 12.0
Encoding: ascii
License: cc0
PoshCode ID: 4015
Published Date: 2013-03-14t16
Archived Date: 2016-08-13t22
---

# connect-accessdb - 

## Description

a set of functions i’ve been working on to allow easy use of access dbs with powershell.  a work in progress, but thought they were at a point where others might find them useful.

## Comments



## Usage



## TODO



## function

`connect-accessdb`

## Code

`#
 #
 
 function Connect-AccessDB ($global:dbFilePath) {
 	
 	if (-not (Test-Path $dbFilePath)) {
 		Write-Error "Invalid Access database path specified. Please supply full absolute path to database file!"
 	}
 	
 	
 	$global:AccessConnection = New-Object -ComObject ADODB.Connection
 	
 	if ((Split-Path $dbFilePath -Leaf) -match [regex]"\.mdb$") {
 		Write-Host "Access 2000-2003 format (MDB) detected!  Using Microsoft.Jet.OLEDB.4.0."
 		$AccessConnection.Open("Provider = Microsoft.Jet.OLEDB.4.0; Data Source= $dbFilePath")
 	}
 	
 	if ((Split-Path $dbFilePath -Leaf) -match [regex]"\.accdb$") {
 		Write-Host "Access 2007 format (ACCDB) detected!  Using Microsoft.Ace.OLEDB.12.0."
 		$AccessConnection.Open("Provider = Microsoft.Ace.OLEDB.12.0; Persist Security Info = False; Data Source= $dbFilePath")
 	} 
 	
 	
 }
 
 function Open-AccessRecordSet ($global:SqlQuery) {
 
 	if ($SqlQuery.length -lt 1) {
 		Throw "Please supply a SQL query for the recordset selection!"
 	}
 	
 	$adOpenStatic = 3
 	$adLockOptimistic = 3
 	
 	$global:AccessRecordSet = New-Object -ComObject ADODB.Recordset
 	
 	$AccessRecordSet.Open($SqlQuery, $AccessConnection, $adOpenStatic, $adLockOptimistic)	
 }
 
 function Get-AccessRecordSetStructure {
 	
 	Write-Output $AccessRecordSet.Fields | Select-Object Name,Attributes,DefinedSize,type
 }
 	
 function Convert-AccessRecordSetToPSObject {
 	
 	$fields = Get-AccessRecordSetStructure
 	
 	$AccessRecordSet.MoveFirst()
 		
 	do {
 		$record = New-Object System.Object
 		
 		foreach ($field in $fields) {
 			$record | Add-Member -type NoteProperty -name $fieldName -value $AccessRecordSet.Fields.Item($fieldName).Value
 		}
 		Write-Output $record
 		
 		$AccessRecordset.MoveNext()
 		
 	} until ($AccessRecordset.EOF -eq $True)
 
 }
 
 function Execute-AccessSQLStatement ($query) {
 	$AccessConnection.Execute($query)
 }
 
 function Convert-AccessTypeCode ([string]$typeCode) {
 	
 	$labelLookupHash = @{"AutoNumber"="3"; "Text"="202"; "Memo"="203"; "Date/Time"="7"; "Currency"="6"; "Yes/No"="11"; "OLE Object"="205"; "Byte"="17"; "Integer"="2"; "Long Integer"="3"; "Single"="4"; "Double"="5"}
 	$codeLookupHash =  @{"3"="AutoNumber"; "202"="Text"; "203"="Memo"; "7"="Date/Time"; "6"="Currency"; "11"="Yes/No"; "205"="OLE Object"; "17"="Byte"; "2"="Integer"; "3"="Long Integer"; "4"="Single"; "5"="Double"}
 	
 	if ($typeCode -match [regex]"^\d{1,3}$") {
 		$valueFound = $codeLookupHash.$typeCode
 		if ($valueFound) {
 			Write-Output $valueFound
 		} else { Write-Output "Unknown" }
 	} else {
 		$valueFound = $labelLookupHash.$typeCode
 		if ($valueFound) {
 			Write-Output $valueFound
 		} else { Write-Output "Unknown" }
 	}
 
 }
 
 function Close-AccessRecordSet {
 	$AccessRecordSet.Close()
 }
 
 function Disconnect-AccessDB {
 	$AccessConnection.Close()
 }
 
 
`

