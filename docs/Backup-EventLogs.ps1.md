---
Author: jeff patton
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2729
Published Date: 2012-06-10t13
Archived Date: 2016-05-23t02
---

# backup-eventlogs - 

## Description

this function backs up eventlogs on a remote computer, where the recordcount for a given log is greater than zero.

## Comments



## Usage



## TODO



## function

`backup-eventlogs`

## Code

`#
 #
 Function Backup-EventLogs
 {
     <#
         .SYNOPSIS
             Backup Eventlogs from remote computer
         .DESCRIPTION
             This function backs up all logs on a Windows computer that have events written in them. This
             log is stored as a .csv file in the current directory, where the filename is the ComputerName+
             Logname+Date+Time the backup was created.
         .PARAMETER ComputerName
             The NetBIOS name of the computer to connect to.
         .EXAMPLE
             Backup-EventLogs -ComputerName dc1
         .NOTES
             May need to be a user with rights to access various logs, such as security on remote computer.
         .LINK
     #>
     
     Param
     (
         [string]$ComputerName
     )
     
     Begin
     {
         $EventLogs = Get-WinEvent -ListLog * -ComputerName $ComputerName
         }
 
     Process
     {
         Foreach ($EventLog in $EventLogs)
         {
             If ($EventLog.RecordCount -gt 0)
             {
                 $LogName = ($EventLog.LogName).Replace("/","-")
                 $BackupFilename = "$($ComputerName)-$($LogName)-"+(Get-Date -format "yyy-MM-dd HH-MM-ss").ToString()+".csv"
                 Get-WinEvent -LogName $EventLog.LogName -ComputerName $ComputerName |Export-Csv -Path ".\$($BackupFilename)"
                 }
             }
         }
 
     End
     {
         Return $?
         }
 }
`

