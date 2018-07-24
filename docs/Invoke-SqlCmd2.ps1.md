---
Author: aaron calderon
Publisher: 
Copyright: 
Email: 
Version: 1.5.3
Encoding: ascii
License: cc0
PoshCode ID: 6341
Published Date: 2016-05-11t14
Archived Date: 2016-05-17t12
---

# invoke-sqlcmd2 () - 

## Description

modeled after sql server 2008 invoke-sqlcmd, but fixes bug in querytimeout, and allows for paramaterized queries and more!

## Comments



## Usage



## TODO



## script

`invoke-sqlcmd2`

## Code

`#
 #
 .SYNOPSIS 
     Runs a T-SQL script. 
 
 .DESCRIPTION 
     Runs a T-SQL script. Invoke-Sqlcmd2 only returns message output, such as the output of PRINT statements when -verbose parameter is specified.
     Paramaterized queries are supported. 
 
     Help details below borrowed from Invoke-Sqlcmd.  Not verified by a SQL expert!
 
 .PARAMETER ServerInstance
     A character string specifying the name of an instance of the Database Engine. For default instances, only specify the computer name: "MyComputer". For named instances, use the format "ComputerName\InstanceName".
 
 .PARAMETER Database
     A character string specifying the name of a database. Invoke-Sqlcmd2 connects to this database in the instance that is specified in -ServerInstance.
 
 .PARAMETER Query
     Specifies one or more queries to be run. The queries can be Transact-SQL (? or XQuery statements, or sqlcmd commands. Multiple queries separated by a semicolon can be specified. Do not specify the sqlcmd GO separator. Escape any double quotation marks included in the string ?). Consider using bracketed identifiers such as [MyTable] instead of quoted identifiers such as "MyTable".
 
 .PARAMETER InputFile
     Specifies a file to be used as the query input to Invoke-Sqlcmd2. The file can contain Transact-SQL statements, (? XQuery statements, and sqlcmd commands and scripting variables ?). Specify the full path to the file.
 
 .PARAMETER Username
     Specifies the login ID for making a SQL Server Authentication connection to an instance of the Database Engine. The password must be specified using -Password. If -Username and -Password are not specified, Invoke-Sqlcmd attempts a Windows Authentication connection using the Windows account running the PowerShell session.
     When possible, use Windows Authentication.
 
 .PARAMETER Password
     Specifies the password for the SQL Server Authentication login ID that was specified in -Username. Passwords are case-sensitive. When possible, use Windows Authentication. Do not use a blank password, when possible use a strong password. For more information, see "Strong Password" in SQL Server Books Online.
     SECURITY NOTE: If you type -Password followed by your password, the password is visible to anyone who can see your monitor. If you code -Password followed by your password in a .ps1 script, anyone reading the script file will see your password. Assign the appropriate NTFS permissions to the file to prevent other users from being able to read the file.
 
 .PARAMETER QueryTimeout
     Specifies the number of seconds before the queries time out.
 
 .PARAMETER ConnectionTimeout
     Specifies the number of seconds when Invoke-Sqlcmd2 times out if it cannot successfully connect to an instance of the Database Engine. The timeout value must be an integer between 0 and 65534. If 0 is specified, connection attempts do not time out.
 
 .PARAMETER As
     Specifies output type - DataSet, DataTable, array of DataRow, or Single Value 
 
 .PARAMETER DBNullToNull
     If specified, array of DataRow results will be converted to PSObject array with no DBNull values.
     Props to Dave Wyatt http://powershell.org/wp/forums/topic/dealing-with-dbnull/
 
 .INPUTS 
     None 
         You cannot pipe objects to Invoke-Sqlcmd2 
 
 .OUTPUTS 
    System.Data.DataTable 
 
 .EXAMPLE 
     Invoke-Sqlcmd2 -ServerInstance "MyComputer\MyInstance" -Query "SELECT login_time AS 'StartTime' FROM sysprocesses WHERE spid = 1" 
     
     This example connects to a named instance of the Database Engine on a computer and runs a basic T-SQL query. 
     StartTime 
     ----------- 
     2010-08-12 21:21:03.593 
 
 .EXAMPLE 
     Invoke-Sqlcmd2 -ServerInstance "MyComputer\MyInstance" -InputFile "C:\MyFolder\tsqlscript.sql" | Out-File -filePath "C:\MyFolder\tsqlscript.rpt" 
     
     This example reads a file containing T-SQL statements, runs the file, and writes the output to another file. 
 
 .EXAMPLE 
     Invoke-Sqlcmd2  -ServerInstance "MyComputer\MyInstance" -Query "PRINT 'hello world'" -Verbose 
 
     This example uses the PowerShell -Verbose parameter to return the message output of the PRINT command. 
     VERBOSE: hello world 
 
 .NOTES 
 Version History 
 v1.0   - Chad Miller - Initial release 
 v1.1   - Chad Miller - Fixed Issue with connection closing 
 v1.2   - Chad Miller - Added inputfile, SQL auth support, connectiontimeout and output message handling. Updated help documentation 
 v1.3   - Chad Miller - Added As parameter to control DataSet, DataTable or array of DataRow Output type 
 v1.4   - Justin Dearing <zippy1981 _at_ gmail.com> - Added the ability to pass parameters to the query.
 v1.4.1 - Paul Bryson <atamido _at_ gmail.com> - Added fix to check for null values in parameterized queries and replace with [DBNull]
 v1.5   - Joel Bennett - add SingleValue output option
 v1.5.1 - RamblingCookieMonster - Added ParameterSets, set Query and InputFile to mandatory
 v1.5.2 - RamblingCookieMonster - Added DBNullToNull switch and code from Dave Wyatt.  Added parameters to comment based help (need someone with SQL expertise to verify these)
 v1.5.3 - Aaron Calderon - Added [switch]$TrustedConnection parameter to allow for connecting with trusted connections to DB.
 #> 
 function Invoke-Sqlcmd2 
 { 
     [CmdletBinding(
         DefaultParameterSetName='Query'
     )] 
     param( 
         [Parameter( Position=0, Mandatory=$true)]
         [string]$ServerInstance, 
     
         [Parameter( Position=1, Mandatory=$false)]
         [string]$Database, 
     
         [Parameter( Position=2,
                     Mandatory=$true,
                     ParameterSetName="Query")]
         [string]$Query, 
     
         [Parameter( Position=2, 
                     Mandatory=$true,
                     ParameterSetName="File")]
         [ValidateScript({test-path $_})]
         [string]$InputFile, 
 
         [Parameter(Position=3, Mandatory=$false)]
         [string]$Username, 
     
         [Parameter(Position=4, Mandatory=$false)]
         [string]$Password, 
     
         [Parameter(Position=5, Mandatory=$false)]
         [Int32]$QueryTimeout=600, 
     
         [Parameter(Position=6, Mandatory=$false)]
         [Int32]$ConnectionTimeout=15, 
     
         [Parameter(Position=7, Mandatory=$false)]
         [ValidateSet("DataSet", "DataTable", "DataRow","SingleValue")]
         [string]$As="DataRow",
     
         [Parameter(Position=8, Mandatory=$false)]
         [System.Collections.IDictionary]$SqlParameters,
 
         [switch]$DBNullToNull,
 
         [Parameter(Mandatory=$false)] [switch]$TrustedConnection 
     ) 
  
     if ($InputFile) 
     { 
         $filePath = $(Resolve-Path $InputFile).path 
         $Query =  [System.IO.File]::ReadAllText("$filePath") 
     } 
  
     $conn = New-Object System.Data.SqlClient.SQLConnection 
     if($TrustedConnection) { $trust = 'True'} else { $trust = 'False'}
       
     if ($TrustedConnection) 
     { $ConnectionString = "Server={0};Database={1};Trusted_Connection={2};Connect Timeout={3}" -f $ServerInstance,$Database,$trust,$ConnectionTimeout } 
     elseif ($Username) 
     { $ConnectionString = "Server={0};Database={1};User ID={2};Password={3};Trusted_Connection=False;Connect Timeout={4}" -f $ServerInstance,$Database,$Username,$Password,$ConnectionTimeout } 
     else 
     { $ConnectionString = "Server={0};Database={1};Integrated Security=True;Connect Timeout={2}" -f $ServerInstance,$Database,$ConnectionTimeout } 
  
     $conn.ConnectionString = $ConnectionString 
      
     if ($PSBoundParameters.Verbose) 
     { 
         $conn.FireInfoMessageEventOnUserErrors=$true 
         $handler = [System.Data.SqlClient.SqlInfoMessageEventHandler] { Write-Verbose "$($_)" } 
         $conn.add_InfoMessage($handler) 
     } 
      
     $conn.Open() 
 
     $cmd = New-Object system.Data.SqlClient.SqlCommand($Query,$conn) 
     $cmd.CommandTimeout=$QueryTimeout
 
     if ($SqlParameters -ne $null)
     {
         $SqlParameters.GetEnumerator() |
             ForEach-Object {
                 If ($_.Value -ne $null)
                 { $cmd.Parameters.AddWithValue($_.Key, $_.Value) }
                 Else
                 { $cmd.Parameters.AddWithValue($_.Key, [DBNull]::Value) }
             } > $null
     }
     
     $ds = New-Object system.Data.DataSet 
     $da = New-Object system.Data.SqlClient.SqlDataAdapter($cmd) 
     
     [void]$da.fill($ds) 
     $conn.Close() 
 
     $cSharp = @'
         using System;
         using System.Data;
         using System.Management.Automation;
 
         public class DBNullScrubber
         {
             public static PSObject DataRowToPSObject(DataRow row)
             {
                 PSObject psObject = new PSObject();
 
                 if (row != null && (row.RowState & DataRowState.Detached) != DataRowState.Detached)
                 {
                     foreach (DataColumn column in row.Table.Columns)
                     {
                         Object value = null;
                         if (!row.IsNull(column))
                         {
                             value = row[column];
                         }
 
                         psObject.Properties.Add(new PSNoteProperty(column.ColumnName, value));
                     }
                 }
 
                 return psObject;
             }
         }
 '@
 
     switch ($As) 
     { 
         'DataSet' 
         {
             $ds
         } 
         'DataTable'
         {
             $ds.Tables
         } 
         'DataRow'
         {
             if(-not $DBNullToNull)
             {
                 $ds.Tables[0]
             }
             else
             {
                 Add-Type -TypeDefinition $cSharp -ReferencedAssemblies 'System.Data','System.Xml'
 
                 foreach ($row in $ds.Tables[0].Rows)
                 {
                     [DBNullScrubber]::DataRowToPSObject($row)
                 }
             }
         }
         'SingleValue'
         {
             $ds.Tables[0] | Select-Object -Expand $ds.Tables[0].Columns[0].ColumnName
         }
     } 
  
`

