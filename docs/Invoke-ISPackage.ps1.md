---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3757
Published Date: 2015-11-11t08
Archived Date: 2015-05-06t18
---

# invoke-ispackage - 

## Description

invoke-ispackage.ps1 script is a wrapper around dtexec.exe to run an ssis package.

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
 Runs an SSIS package from an SQL package store.
 .DESCRIPTION
 Invoke-ISPackage.ps1 script is a wrapper around dtexec.exe to run an SSIS package.
 .EXAMPLE
 PowerShell.exe -File "C:\Scripts\Invoke-ISPackage.ps1" -PackagePath "SQLPSX\sqlpsx1" -ServerInstance "Z001\SQL1"
 This example connects to SSIS Server  and runs the an SSIS package on Z001\SQL1
 .NOTES
 Version History
 v1.0   - Chad Miller - 6/14/2012 - Initial release
 #>
 param(
 [Parameter(Position=0, Mandatory=$true)]
 [string]
 $PackagePath,
 [Parameter(Position=1, Mandatory=$true)]
 [string]
 $ServerInstance,
 [Parameter(Position=2, Mandatory=$false)]
 [ValidateNotNullOrEmpty()]
 [ValidateScript({Test-Path "$_"})]
 [string]
 $CommandFile,
 [Parameter(Position=3, Mandatory=$false)]
 [ValidateNotNullOrEmpty()]
 [string]
 $Application="Invoke-ISPackage.ps1",
 [Parameter(Position=4, Mandatory=$false)]
 [switch]
 $Use32Bit
 )
 
 
 New-EventLog -LogName Application -Source $Application -EA SilentlyContinue
 $Error.Clear()
 
 $ErrorActionPreference = "Stop"
 $exitCode = @{
 0="The package executed successfully."
 1="The package failed."
 3="The package was canceled by the user."
 4="The utility was unable to locate the requested package. The package could not be found."
 5="The utility was unable to load the requested package. The package could not be loaded."
 6="The utility encountered an internal error of syntactic or semantic errors in the command line."}
 
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
 
 $options = @"
 /Rep E /DTS "$PackagePath" /Conf "c:\SSISconfig\$ServerInstance\SSISconnection.dtsConfig"
 "@
 
 if ($CommandFile) {
     $options += " /Com `"$CommandFile`""
 }
 
 $paths = [Environment]::GetEnvironmentVariable("Path", "Machine") -split ";"
 
 if ($Use32Bit) {
     $dtexec = $paths | where { $_ -like "*Program Files(x86)\Microsoft SQL Server\*\DTS\Binn\" }
     $dtexec += 'dtexec.exe'
 }
 
 if ($dtexec -eq $null) {
     $dtexec = $paths | where { $_ -like "*Program Files\Microsoft SQL Server\*\DTS\Binn\" }
     $dtexec += 'dtexec.exe'
 }
 
 write-verbose "$dtexec $options"
 $Context = "servername = $ServerInstance`npath = $PackagePath"
 $msg = "ApplicationStartEvent"
 
 Write-Message -Severity Information -Category $events.ApplicationStartEvent -Eventid 1 -ShortMessage $msg -Context $Context
 
 try {
     
     $result = & $dtexec $options
     $result = $result -join "`n"
 
     if ($lastexitcode -eq 0) {
         $msg = "ApplicationStopEvent"
         Write-Message -Severity Information -Category $events.ApplicationStopEvent -Eventid 99 -ShortMessage $msg -Context $Context
     }
     else {
         throw $exitcode[$lastexitcode]
     }
 }
 catch {
     $result = "$_ `n$result"
     if ($result -and $result.Length -gt $MaxErrorMsgLen) {
         $result = $result.SubString($result.Length - $MaxErrorMsgLen)
     }
     Write-Verbose "DtExecException"
     Write-Message -Severity Error -Category  $events.Get_Item("ConfigurationException") -Eventid 334 -ShortMessage "ConfigurationException: $result" -Context $Context
     throw "$result `n $("Failed to run PackagePath {0} to ServerInstance {1}" -f $PackagePath,$ServerInstance)"
 }
`

