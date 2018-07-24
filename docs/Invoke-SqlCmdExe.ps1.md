---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3756
Published Date: 2013-11-11t06
Archived Date: 2017-04-21t01
---

# invoke-sqlcmdexe - 

## Description

invoke-sqlcmdexes.ps1 script is a wrapper around sqlcmd.exe to run a t-sql query or stored procedure and optionally outputs to a file.

## Comments



## Usage



## TODO



## script

`write-message`

## Code

`#
 #
 <#
 .SYNOPSIS
 Runs a T-SQL Query and optional outputs results to a file.
 .DESCRIPTION
 Invoke-SqlCmdExes.ps1 script is a wrapper around sqlcmd.exe to run a T-SQL query or stored procedure and optionally outputs to a file.
 .EXAMPLE
 PowerShell.exe -File "C:\Scripts\Invoke-SqlCmdExe.ps1" -ServerInstance "Z001\sql1" -Database accounting -Query "EXEC usp_accounts '12445678'"
 This example connects to Z001\sql1.accounting and executes a stored procedure which does not return a result set
 .EXAMPLE
 PowerShell.exe -File "C:\Scripts\Invoke-SqlCmdExe.ps1" -ServerInstance "Z001\sql1" -Database accounting -Query "SET NOCOUNT ON; SELECT * FROM dbo.vw_accounts" -FilePath "C:\Scripts\accounts.txt" -SqlCmdOptions '-h-1 -s"|" -w 8000'
 This example connects to Z001\sql1.accounting and selects the records from the vw_accounts view, the data is outputed to a pipe delimited file with additional options
 .EXAMPLE
 PowerShell.exe -File "C:\Scripts\Invoke-SqlCmdExe.ps1" -ServerInstance "Z001\sql1" -Database accounting -Query "EXEC usp_accounts '12445678'" -FilePath "C:\Scripts\accounts.txt" -SqlCmdOptions '-h-1 -s","'
 This example connects to Z001\sql1.accounting and selects the records from the vw_accounts view, the data is outputed to a CSV file
 .NOTES
 Version History
 v1.0   - Chad Miller - 5/3/2012 - Initial release
 #>
 param(
 [Parameter(Position=0, Mandatory=$true)]
 [string]
 $ServerInstance,
 [Parameter(Position=1, Mandatory=$true)]
 [string]
 $Database,
 [Parameter(Position=2, Mandatory=$true)]
 [string]
 $Query,
 [Parameter(Position=3, Mandatory=$false)]
 [ValidateNotNullOrEmpty()]
 [string]
 $Application="Invoke-SqlCmdExe.ps1",
 [Parameter(Position=4, Mandatory=$false)]
 [ValidateNotNullOrEmpty()]
 [ValidateScript({Test-Path $([system.io.path]::GetDirectoryName("$_"))})]
 [string]
 $FilePath,
 [Parameter(Position=5, Mandatory=$false)]
 [ValidateNotNullOrEmpty()]
 [string]
 $SqlCmdOptions
 )
 
 
 New-EventLog -LogName Application -Source $Application -EA SilentlyContinue
 $Error.Clear()
 
 $events = @{"ApplicationStartEvent" = "31101"; "ApplicationStopEvent" = "31104"; "DatabaseException" = "31725"; "ConfigurationException" = "31705";"BadDataException" = "31760"}
 $msg =$null
 $MaxErrorMsgLen = 3000
 
 #######################
 function Write-Message{
 Param([string]$Severity,[string]$Category,[string]$Eventid,[string]$ShortMessage=$null,[string]$Context=$null)
    
    $msg = @"
 Severity: $Severity
 Category: $Category
 EventID: $Eventid
 Short Message: $ShortMessage
 Context: $Context
 "@
    $msg = $msg -replace "'"
      
   Write-EventLog -LogName Application -Source $Application -EntryType $Severity -EventId $Eventid -Category $Category -Message $msg              
 
 
 #######################
 #######################
 
 $Options = @"
 -S"$ServerInstance" -d "$Database" -Q "$Query"
 "@
 
 if ($FilePath) {
     $Options += @"
  -o "$FilePath"
 "@
 }
 
 if ($SqlCmdOptions) {
     $Options += " $SqlCmdOptions"
 }
    
 Write-Verbose "sqlcmd.exe $Options"
 
 $Context = "ServerInstance\Database = $ServerInstance\$Database`nQuery = $Query"
 $msg = "ApplicationStartEvent"
 
 Write-Message -Severity Information -Category $events.ApplicationStartEvent -Eventid 1 -ShortMessage $msg -Context $Context
     
 try {
     if ($FilePath) {
        $exitCode = (Start-Process -FilePath "sqlcmd.exe" -ArgumentList @"
 $Options
 "@ -Wait -NoNewWindow -Passthru).ExitCode
     }
     else {
         $tempFile = [io.path]::GetTempFileName()
         $exitCode = (Start-Process -FilePath "sqlcmd.exe" -ArgumentList @"
 $Options
 "@ -Wait -NoNewWindow -RedirectStandardOutput $tempFile -Passthru).ExitCode
     }
 
     if ($ExitCode -eq 0) {
         $msg = "ApplicationStopEvent"
         Write-Message -Severity Information -Category $events.ApplicationStopEvent -Eventid 99 -ShortMessage $msg -Context $Context
     }
     else {
         throw
     }
 }
 catch [InvalidOperationException] {
     $Exception = "{0}, {1}" -f  $_.Exception.GetType().FullName,$( $_.Exception.Message -replace "'" )
     Write-Verbose "InvalidOperationException"
     Write-Message -Severity Error -Category $events.ConfigurationException -Eventid 99 -ShortMessage "ConfigurationException: $Exception" -Context $Context
     throw $_
 }
 catch {
     if ($FilePath) {
         $Exception = [System.IO.File]::ReadAllText("$FilePath")
     }
     elseif ($tempFile) {
         $Exception = [System.IO.File]::ReadAllText("$tempfile")
     }
 
     if ($Exception -and $Exception.Length -gt $MaxErrorMsgLen) {
         $Exception = $Exception.SubString($Exception.Length - $MaxErrorMsgLen)
     }
     Write-Verbose "SqlcmdException"
     Write-Message -Severity Error -Category $events.DatabaseException -Eventid 99 -ShortMessage "DatabaseException: $Exception" -Context $Context
     throw $Exception
 }
 finally {
     if ($tempFile) {
         remove-item $tempFile
     }
 }
`

