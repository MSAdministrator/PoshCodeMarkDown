---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4464
Published Date: 2014-09-12t09
Archived Date: 2014-08-18t21
---

# show-sqlprocesses - show-sqlprocesses.ps1

## Description

#

## Comments



## Usage



## TODO



## function

`show-sqlprocesses`

## Code

`#
 #
   #############################################################################################
 #
 #
 
 Function Show-SQLProcesses ($SQLServer)
 
 {
 $server = new-object "Microsoft.SqlServer.Management.Smo.Server" $SQLServer
 $Server.EnumProcesses()|Select Spid,BlockingSpid, Login, Host,Status,Program,Command,Database,Cpu,MemUsage |Format-Table -wrap -auto
 
 $OUTPUT= [System.Windows.Forms.MessageBox]::Show("Do you want to Kill a process?" , "Question" , 4) 
 
 if ($OUTPUT -eq "YES" ) 
 {
 
 $spid = Read-Host "Which SPID?"
 $Server.KillProcess($Spid)
 
 
 } 
 else 
 { 
 
 }
 
 }
`

