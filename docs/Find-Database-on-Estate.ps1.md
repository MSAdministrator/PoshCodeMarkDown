---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4435
Published Date: 2014-09-01t11
Archived Date: 2017-05-13t16
---

# find database on estate - find-database.ps1

## Description

#############################################################################################

## Comments



## Usage

find-database dbname

## TODO



## function

`find-database`

## Code

`#
 #
 
  #############################################################################################
 #
 #
 
 
 Function Find-Database ([string]$Search)
 {
 
 [System.Reflection.Assembly]::LoadWithPartialName('Microsoft.SqlServer.SMO') | out-null
 
 
 $servers = Get-Content 'c:\temp\sqlservers.txt'
 
 $ht =@{}
 $b = 0
 
 				Write-Host "#################################"
 				Write-Host "Searching for $DatabaseNameSearch "  
 				Write-Host "#################################"  
 
 $DatabaseNameSearch = $search.ToLower()                                   
 
                     
 foreach($server in $servers)
 {
 	$srv = New-Object ('Microsoft.SqlServer.Management.Smo.Server') $server
     
 	foreach($database in $srv.Databases)
 	{
     $databaseName = $database.Name.ToLower()
 
     	if($databaseName.Contains($DatabaseNameSearch))
         {
 
 		$DatabaseNameResult = $database.name
         $Key = "$Server             $DatabaseNameResult"
         $ht.add($Key ,$b)
         $b = $b +1
         }
 
     }        
 }
 
 
 $Results = $ht.GetEnumerator() | Sort-Object Name|Select Name
 $Resultscount = $ht.Count
 
 if ($Resultscount -gt 0)
                         {
 
                                 foreach($R in $Results)
                                     {
 				                    Write-Host $R.Name 
                                     }
                         }
 
 Else
 {
 
 
 }             
 
 }
`

