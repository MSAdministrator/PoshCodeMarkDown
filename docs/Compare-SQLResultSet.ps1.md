---
Author: josh feierman
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3709
Published Date: 2012-10-23t07
Archived Date: 2012-10-28t00
---

# compare-sqlresultset - 

## Description

a function to compare result sets from sql server queries. i wrote it to help with ensuring that original and modified code returned the same results. currently only works with similarly shaped results sets (i.e. those with the same number of columns).

## Comments



## Usage



## TODO



## script

`compare-sqlresultset`

## Code

`#
 #
 <#
 .SYNOPSIS
   Compares two result sets from SQL queries for differences.
 .DESCRIPTION
   Compares the result sets from two SQL queries and outputs the
   differences.
   
   Currently the function only handles similarly shaped result sets. That is,
   result sets with the same number and names of columns. Functionality will be
   added in the future to allow for comparison and differencing of disparate result
   sets.
 .PARAMETER ServerName1
   The name of the server on which the first query should be executed.
 .PARAMETER DatabaseName1
   The name of the database on which the first query should be executed.
 .PARAMETER Query1
   The first SQL query to be executed.
 .PARAMETER ServerName2
   The name of the server on which the second query should be executed.
 .PARAMETER DatabaseName2
   The name of the database on which the second query should be executed.
 .PARAMETER Query2
   The second SQL query to be executed.
 .EXAMPLE
   Compare-SQLResultSet -ServerName1 myServer1 -DatabaseName1 myDatabase -Query1 "exec dbo.myproc" -ServerName2 myServer2 -DatabaseName2 myDatabase -Query2 "exec dbo.myproc_changed"
 
   Could be used to compare two result sets from the same server and database, but two different
   procedures. Useful for comparing old and new code.
 .NOTES
   Author: Josh Feierman
   Version: 1
 
 #>
 function Compare-SQLResultSet
 {
   [Cmdletbinding()]
   param
   (
       [parameter(mandatory=$true)]
       [String]$ServerName1,
       [parameter(mandatory=$true)]
       [String]$DatabaseName1,
       [parameter(mandatory=$true)]
       [String]$Query1,
       [parameter(mandatory=$true)]
       [String]$ServerName2,
       [parameter(mandatory=$true)]
       [String]$DatabaseName2,
       [parameter(mandatory=$true)]
       [String]$Query2
   )
   try
   {
       if (-not (Get-Module -Name SQLPS -ListAvailable))
       {
           Write-Warning "The SQLPS module is not installed, please obtain and install it."
           return
       }
       Import-Module SQLPS
   
       $resultSet1 = Invoke-Sqlcmd -ServerInstance $ServerName1 -Database $DatabaseName1 -Query $Query1
       $resultSet2 = Invoke-Sqlcmd -ServerInstance $ServerName2 -Database $DatabaseName2 -Query $Query2
   
       $firstCount = $resultSet1.Count
       $secondCount = $resultSet2.Count
   
       if ($firstCount -gt $secondCount) {$totalCount = $firstCount}
       elseif ($secondCount -gt $firstCount) {$totalCount = $secondCount}
       else {$totalCount = $firstCount}
   
       for ($counter = 0; $counter -lt $totalCount; $counter ++)
       {
           if ($counter -lt $firstCount)
           {
               $firstRow = $resultSet1[$counter]
           }
           else
           {
               $firstRow = $null
           }
           if ($counter -lt $secondCount)
           {
               $secondRow = $resultSet2[$counter]
           }
           else
           {
               $secondRow = $null
           }
   
           $compareObject = New-Object PSObject -Property @{
               RowNumber = $counter + 1
               ColumnName = $null
               FirstValue = $null
               SecondValue = $null
           }
   
           $properties = Get-Member -inputObject $firstRow -MemberType Properties
   
           foreach ($property in $properties)
           {
               $firstValue = $firstRow.Item($property.Name)
               $secondValue = $secondRow.Item($property.Name)
   
               if ($firstValue -ne $secondValue)
               {
                   $compareObject.ColumnName = $property.Name
                   $compareObject.FirstValue = $firstValue
                   $compareObject.SecondValue = $secondValue
   
                   Write-Output $compareObject
               }
           }
       }
   }
   catch
   {
       Write-Warning $_.Exception.Message
   }
 
 }
`

