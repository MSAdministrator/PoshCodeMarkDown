---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2732
Published Date: 2011-06-13t18
Archived Date: 2011-10-25t12
---

# test-sqlconnection - 

## Description

a series of functions to verify agent, sql, ssis, smb, wmi services or connections. useful when troubleshooting data center firewall configuration.

## Comments



## Usage



## TODO



## script

`test-ping`

## Code

`#
 #
 #######################
 <#
 Version History
 v1.0   - Chad Miller - Initial release
 v1.1   - Chad Miller - Fixed issues, added colorized HTML output
 #>
 
 
 Add-Type -AssemblyName System.Xml.Linq
 $Script:CMServer = 'MyServer'
 
 #######################
 function Test-Ping
 {
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] [string[]]$ComputerName)
 
     process
     { 
         $ComputerName | foreach {$result=Test-Connection -ComputerName $_ -Count 1 -Quiet; 
         new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args=$_;Result=$result;Message=$null}}
     }
 
 
 #######################
 function Test-Wmi
 {
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] [string[]]$ComputerName)
 
     process
     { 
         try { 
             $ComputerName | foreach {$name=$_; Get-WmiObject -ComputerName $name -Class Win32_ComputerSystem -ErrorAction 'Stop' | out-null; 
             new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$true;Message=$null}}
             }
         catch { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message="$($_.ToString())"} }
     }
 
 
 #######################
 function Test-Port
 {
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] [string[]]$ComputerName
     ,[Parameter(Mandatory=$true)] [int32]$Port)
      
     Process {
         try {
             $Computername | foreach {
             $sock = new-object System.Net.Sockets.Socket -ArgumentList $([System.Net.Sockets.AddressFamily]::InterNetwork),$([System.Net.Sockets.SocketType]::Stream),$([System.Net.Sockets.ProtocolType]::Tcp); $name=$_; `
             $sock.Connect($name,$Port)
             new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$true;Message=$null}
             $sock.Close()}
          
         }
         catch { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message="$($_.ToString())"} }
     }
 
 
 #######################
 function Test-SMB
 {
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] [string[]]$ComputerName)
 
     process
     { Test-Port $ComputerName 445 }
     
 
 #######################
 function Test-SSIS
 {
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] [string[]]$ComputerName
      ,[Parameter(Mandatory=$true)] [ValidateSet("2005", "2008")] [string]$Version
     )
 
 
     Begin {
         $ErrorAction = 'Stop'
         if ($Version -eq 2008)
         { add-type -AssemblyName "Microsoft.SqlServer.ManagedDTS, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91" }
         else
         { add-type -AssemblyName "Microsoft.SqlServer.ManagedDTS, Version=9.0.242.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91" }
     }
 
     process
     { 
         $ComputerName | foreach {
         $name=$_; `
         if ((test-SSISService -ComputerName $name).Result)
         {
             try { 
                     
                     $app = new-object ("Microsoft.SqlServer.Dts.Runtime.Application")
                     $out = $null
                     $app.GetServerInfo($name,[ref]$out) | out-null
                     new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$true;Message=$null}
             }
             catch { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message="$($_.ToString())"}} 
         }
         else
         { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message='SSIS Not Running'} }
         }
     }
 
 
 #######################
 function Test-SQL
 {
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] [string[]]$ServerInstance)
 
     process {
         $ServerInstance | foreach {
         $name = $_; `
             if ((test-Ping $(ConvertTo-ComputerName $name)).Result)
             {
                 $connectionString = "Data Source={0};Integrated Security=true;Initial Catalog=master;Connect Timeout=3;" -f $name
                 $sqlConn = new-object ("Data.SqlClient.SqlConnection") $connectionString                                          
                 try { 
                     $sqlConn.Open()
                     new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$true;Message=$null}
                 }
                 catch { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message="$($_.ToString())"} }
                 finally { $sqlConn.Dispose() }
             }
             else
             { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message='Host Unreachable'} }
         }
     }
 
 
 #######################
 function Test-AgentService
 {
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] [string[]]$ComputerName)
 
     process
     { 
         try {
             $ComputerName | foreach {
             $name=$_; `
             if (Get-WmiObject -Class Win32_Service -ComputerName $name -Filter {Name Like 'SQLAgent%' and State='Stopped'} -ErrorAction 'Stop') {
             new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message=$null} }
             else {
             new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$true;Message=$null} }
             }
         }
         catch { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message="$($_.ToString())"} }
     }
 
 
 #######################
 function Test-SqlService
 {
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] [string[]]$ComputerName)
 
     process
     { 
         try {
             $ComputerName | foreach {
             $name=$_; `
             if (Get-WmiObject -ComputerName $name `
             -query "select Name,State from Win32_Service where (NOT Name Like 'MSSQLServerADHelper%') AND (Name Like 'MSSQL%' OR Name Like 'SQLServer%') And State='Stopped'" `
             -ErrorAction 'Stop')
             { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message=$null} }
             else
             { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$true;Message=$null} }
             }
         }
         catch { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message="$($_.ToString())"} }
     }
    
 
 #######################
 function Test-SSISService
 {
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] [string[]]$ComputerName)
 
     process
     { 
         try {
             $ComputerName | foreach {
             $name=$_; `
             if (Get-WmiObject -Class Win32_Service -ComputerName $ComputerName -Filter {Name Like 'MsDtsServer%' And State='Stopped'} -ErrorAction 'Stop')
             { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message=$null} }
             else
             { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$true;Message=$null} }
             }
         }
         catch { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message="$($_.ToString())"} }
     }
 
 
 #######################
 <#
 .SYNOPSIS
 Runs a T-SQL script.
 .DESCRIPTION
 Runs a T-SQL script. Invoke-Sqlcmd2 only returns message output, such as the output of PRINT statements when -verbose parameter is specified
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
 #>
 function Invoke-Sqlcmd2
 {
     [CmdletBinding()]
     param(
     [Parameter(Position=0, Mandatory=$true)] [string]$ServerInstance,
     [Parameter(Position=1, Mandatory=$false)] [string]$Database,
     [Parameter(Position=2, Mandatory=$false)] [string]$Query,
     [Parameter(Position=3, Mandatory=$false)] [string]$Username,
     [Parameter(Position=4, Mandatory=$false)] [string]$Password,
     [Parameter(Position=5, Mandatory=$false)] [Int32]$QueryTimeout=600,
     [Parameter(Position=6, Mandatory=$false)] [Int32]$ConnectionTimeout=15,
     [Parameter(Position=7, Mandatory=$false)] [ValidateScript({test-path $_})] [string]$InputFile,
     [Parameter(Position=8, Mandatory=$false)] [ValidateSet("DataSet", "DataTable", "DataRow")] [string]$As="DataRow"
     )
 
     if ($InputFile)
     {
         $filePath = $(resolve-path $InputFile).path
         $Query =  [System.IO.File]::ReadAllText("$filePath")
     }
 
     $conn=new-object System.Data.SqlClient.SQLConnection
      
     if ($Username)
     { $ConnectionString = "Server={0};Database={1};User ID={2};Password={3};Trusted_Connection=False;Connect Timeout={4}" -f $ServerInstance,$Database,$Username,$Password,$ConnectionTimeout }
     else
     { $ConnectionString = "Server={0};Database={1};Integrated Security=True;Connect Timeout={2}" -f $ServerInstance,$Database,$ConnectionTimeout }
 
     $conn.ConnectionString=$ConnectionString
     
     if ($PSBoundParameters.Verbose)
     {
         $conn.FireInfoMessageEventOnUserErrors=$true
         $handler = [System.Data.SqlClient.SqlInfoMessageEventHandler] {Write-Verbose "$($_)"}
         $conn.add_InfoMessage($handler)
     }
     
     $conn.Open()
     $cmd=new-object system.Data.SqlClient.SqlCommand($Query,$conn)
     $cmd.CommandTimeout=$QueryTimeout
     $ds=New-Object system.Data.DataSet
     $da=New-Object system.Data.SqlClient.SqlDataAdapter($cmd)
     [void]$da.fill($ds)
     $conn.Close()
     switch ($As)
     {
         'DataSet'   { Write-Output ($ds) }
         'DataTable' { Write-Output ($ds.Tables) }
         'DataRow'   { Write-Output ($ds.Tables[0]) }
     }
 
 
 #######################
 function Test-DatabaseOnline
 {
     param([Parameter(Mandatory=$true, ValueFromPipeline = $true)] [string[]]$ServerInstance)
     
     begin
     {
 $query = @"
 SELECT name
 FROM sysdatabases
 WHERE DATABASEPROPERTYEX(name,'Status') <> 'ONLINE'
 "@
     }
     process
     { 
         try {
             $ServerInstance | foreach {
             $name=$_; `
             $out = Invoke-Sqlcmd2 -ServerInstance $name -Database master -Query $query -ConnectionTimeout 5 | foreach {$_.name}
             if ($out)
             {$out = $out -join ","; new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message=$out} }
             else
             { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$true;Message=$null} }
             }
         }
         catch { new-object psobject -property @{Test="$($myinvocation.mycommand.name)";Args="$name";Result=$false;Message="$($_.ToString())"} }
     }
 
 
 #######################
 filter ConvertTo-ComputerName
 {
     param ($ServerInstance)
      
     if ($_)
     { $ServerInstance = $_ }
 
     $ServerInstance -replace "\\.*|,.*"
     
 
 
 #######################
 filter ConvertTo-NamePortNumber
 {
     param ($ServerInstance)
      
     if ($_)
     { $ServerInstance = $_ }
 
     $work = $ServerInstance  -split ","
     
     if ($work[1])
     { new-object psobject -Property @{ComputerName=$work[0];PortNumber=$work[1]} }
 
 
 #######################
 function Get-CMServer
 {
     param($ServerInstance,$GroupName,[switch]$UsePort)
 
 $query = @"
 FROM msdb.dbo.sysmanagement_shared_registered_servers s
 JOIN msdb.dbo.sysmanagement_shared_server_groups g
 ON s.server_group_id = g.server_group_id
 WHERE 1 = 1
 "@
 
 if ($UsePort) {
 $query = "SELECT DISTINCT s.server_name AS 'name'`n" + $query + "`nAND PATINDEX('%,%', s.server_name) <> 0"
 }
 else {
 $query = "SELECT DISTINCT s.name`n" + $query
 
 }
 if ($GroupName) {
 $query = $query + "`nAND g.name = '$GroupName'"
 }
 
 Invoke-SqlCmd2 -ServerInstance $ServerInstance -Database msdb -Query $query | foreach {$_.name}
 
 
 #######################
 filter Get-SqlVersion2
 {
     param($ServerInstance)
 
 $query = @"
 select CAST(SERVERPROPERTY('ServerName') AS varchar(128)) AS 'server_name',
 CASE CONVERT(int, LEFT(CONVERT(varchar(10),SERVERPROPERTY('ProductVersion')),1))
 WHEN 1 THEN '2008'
 WHEN 9 THEN '2005'
 WHEN 8 THEN '2000'
 END AS 'version'
 "@
     
     if ($_)
     { $ServerInstance = $_ }
 
 Invoke-SqlCmd2 -ServerInstance $ServerInstance -Database master -Query $query -ConnectionTimeout 5
 
 
 #######################
 function Test-Main
 {
 [CmdletBinding()]
     param(
     [Parameter(Position=0, Mandatory=$true)] [ValidateSet("Ping","WMI","SMB","Port","SQL","Database","SSIS","Agent")] [string]$Test,
     [Parameter(Position=1, Mandatory=$false)] [string]$GroupName,
     [Parameter(Position=2, Mandatory=$false)] [string]$SqlVersion='2008'
     )
 
 switch ($test)
 {
         'Ping' { $html = Get-CMServer $Script:CMServer  $GroupName | ConvertTo-ComputerName | Test-Ping | ConvertTo-Html }
         'WMI' { $html = Get-CMServer $Script:CMServer $GroupName  | ConvertTo-ComputerName | Test-WMI | ConvertTo-Html }
         'SMB' { $html = Get-CMServer $Script:CMServer $GroupName  | ConvertTo-ComputerName | Test-SMB | ConvertTo-Html }
         'Port' { $html = Get-CMServer $Script:CMServer $GroupName  -UsePort | ConvertTo-NamePortNumber | foreach {Test-Port $_.ComputerName $_.PortNumber} | ConvertTo-Html }
         'SQL' { $html = Get-CMServer $Script:CMServer $GroupName  | Test-SQL | ConvertTo-Html }
         'Database' { $html = Get-CMServer $Script:CMServer $GroupName  | Test-DatabaseOnline | ConvertTo-Html }
         'SSIS' { $html = Get-CMServer $Script:CMServer $GroupName  | Get-SqlVersion2 | where {$_.version -eq $SqlVersion } | foreach {$_.server_name} | ConvertTo-ComputerName | Test-SSIS  -Version $SqlVersion | ConvertTo-Html }
         'Agent' { $html = Get-CMServer $Script:CMServer  $GroupName | ConvertTo-ComputerName | Test-AgentService | ConvertTo-Html }
 }
 
 $xml = [System.Xml.Linq.XDocument]::Parse("$html")
 
 $index = (($xml.Descendants("{http://www.w3.org/1999/xhtml}th") | Where-Object { $_.Value -eq "Result" }).NodesBeforeSelf() | Measure-Object).Count
 
 switch($xml.Descendants("{http://www.w3.org/1999/xhtml}td") | Where { ($_.NodesBeforeSelf() | Measure).Count -eq $index } ) {
    {'True' -eq $_.Value } { $_.SetAttributeValue( "style", "background: green;"); continue } 
    {'False' -eq $_.Value } { $_.SetAttributeValue( "style", "background: red;"); continue } 
    
 }
 $xml.Save("$pwd/$test.html")
 
 ii .\$test.html
`

