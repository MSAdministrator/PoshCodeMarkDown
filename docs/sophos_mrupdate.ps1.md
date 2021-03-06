---
Author: tojo2000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3160
Published Date: 2012-01-10t17
Archived Date: 2012-01-15t04
---

# sophos_mrupdate.ps1 - 

## Description

a script for updating the message relay server used by a sophos client.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
    Updates the Sophos Message Relay configuration.
 .DESCRIPTION
    Points the local Message Router service to a new parent relay server.
 .PARAMETER mrinit
    The mrinit.conf file to use.
 .PARAMETER loglevel
    The current loglevel (used by the Log function).  
    Defaults to 'information' (debug messages will not be printed).
    Valid values are 'debug', 'information', 'warning', and 'error'.
 .EXAMPLE
    sophos_mrupdate.ps1 -mrinit AMW_mrinit.conf
 #>
 param([Parameter(Mandatory=$true)][string]$mrinit,
       [Parameter(Mandatory=$false)][string]$loglevel=1)
 
 
 $MSG_DEBUG = 0
 $MSG_INFORMATION = 1
 $MSG_WARNING = 2
 $MSG_ERROR = 3
 $MSG_SEVERITY = @('debug', 'information', 'warning', 'error')
 
 if ($MSG_SEVERITY -notcontains $loglevel.ToLower()) {
   Write-Error ('Invalid loglevel!  ' +
                'Must be debug, information, warning, or error.')
   exit 1
 } else {
   foreach ($index in (0..($MSG_SEVERITY.Count - 1))) {
     if ($MSG_SEVERITY[$index] -eq $loglevel) {
       $loglevel = $index
     }
   }
 }
 
 $LOG_FILE = 'c:\windows\temp\sophos_mrupdate.log'
 $SMTP_TO = 'sophos-virusalerts@company.com'
 $SMTP_SERVER = 'smtp.company.com'
 $SMTP_SUBJECT = 'Sophos MRUpdate Error'
 $SMTP_FROM = 'sophos-mrupdate@company.com'
 
 
 function Log {
 <#
 .SYNOPSIS
    Writes a message to the Log.
 .DESCRIPTION
   Logs a message to the logfile if the severity is higher than $loglevel.
 .PARAMETER severity
    The severity of the message.  Can be Information, Warning, or Error.
    Use the $MSG_XXXX constants.
    Note that Error will halt the script and send an email.
 .PARAMETER message
    A string to be printed to the log.
 .EXAMPLE
    Log $MSG_ERROR "Something has gone terribly wrong!"
 #>
   param([Parameter(Mandatory=$true)][int]$severity,
         [Parameter(Mandatory=$true)][string]$message,
         [Parameter()][switch]$sendmail)
 
   if ($severity -ge $loglevel) {
     $timestamp = Get-Date -Format 'yyyy-MM-dd hh:mm:ss'
     $output = ("$timestamp`t$($env:computername)`t" +
                "$($MSG_SEVERITY[$severity])`t$message")
     Write-Output $output >> $LOG_FILE
 
 
     if (($severity -ge $MSG_ERROR) -or $sendmail) {
       Send-MailMessage -To $SMTP_TO `
                        -SmtpServer $SMTP_SERVER `
                        -Subject "$SMTP_SUBJECT on $($env:computername)" `
                        -Body $output `
                        -From $SMTP_FROM
       exit 1
     }
   }
 }
 
 
 
 $CURRENT_PATH = $pwd
 $RMS_PATH = 'C:\Program Files (x86)\Sophos\Remote Management System\'
 
 if (-not (Test-Path $RMS_PATH)) {
   $RMS_PATH = 'C:\Program Files\Sophos\Remote Management System\'
   
   if (-not (Test-Path $RMS_PATH)) {
     Log $MSG_ERROR "The path '$RMS_PATH' could not be found!"
   }
 }
 
 Log $MSG_DEBUG "Copying file '$mrinit'..."
 
 if (Test-Path ".\$mrinit") {
   try {
     Copy-Item -Path ".\$mrinit" -Destination ($RMS_PATH + 'mrinit.conf') -Force
   } catch {
     LOG $MSG_ERROR "Unable to copy $mrinit to the RMS directory!" 
   }
 } else {
   LOG $MSG_ERROR "File '$mrinit' missing!  Check the SCCM package."
   exit 1
 }
 
 Log $MSG_DEBUG "Changing directory to $RMS_PATH..."
 cd $RMS_PATH
 
 Log $MSG_DEBUG 'Renaming mrinit.conf.orig to mrinit.conf.orig.old'
 
 try {
   Rename-Item '.\mrinit.conf.orig' '.\mrinit.conf.orig.old' `
               -Force `
               -ea SilentlyContinue
 } catch {
   Log $MSG_ERROR "Unable to rename the file 'mrinit.orig'!"
 }
 
 .\ClientMRInit.exe
 
 net stop "Sophos Message Router"
 net start "Sophos Message Router"
 
 cd $CURRENT_PATH
`

