---
Author: pedro genil
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3909
Published Date: 2013-01-18t09
Archived Date: 2013-01-23t06
---

# backup exchange 2007 - 

## Description

comprobamos que sg del mailbox lleva mas de x dias sin hacerse, y de ese lanzamos un full.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 #######################################################################################
 ######################################################################################
 
 If ((Get-PSSnapin | where {$_.Name -match "Exchange.Management"}) -eq $null)
 {
 	Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Admin
 }
 $lista_incremental = "" 
 $lista_full = ""
 $oldest_full = $(Get-Date).AddDays(4) 
 $server = "MAILBOX" 
 
    foreach ($database in Get-MailboxDatabase -Server $server -Status | where { $_.storagegroupname -notlike "TSMRSG" }) 
    { 
          if (-not $database.Recovery) 
          { 
             if ($database.Mounted) 
             { 
                if (-not $database.BackupInProgress) 
                { 
 
                   if ($database.LastFullBackup -lt $oldest_full) 
                   { 
                      $lista_full = "`"" + $($database.StorageGroupname) + "`"" 
                      $oldest_full = $database.LastFullBackup 
                   } 
                   
                } 
             } 
          } 
    } 
 
    foreach ($database in Get-MailboxDatabase -Server $server -Status | where { $_.storagegroupname -notlike "TSMRSG" }) 
    { 
          if (-not $database.Recovery) 
          { 
             if ($database.Mounted) 
             { 
                if (-not $database.BackupInProgress) 
                { 
 			
                   if (-not $database.CircularLoggingEnabled) 
                   { 
 			if ($lista_full -ne "`"" + $($database.StorageGroupname) + "`"")
 			{
                      		if ($lista_incremental.Length -gt 0) 
                      			{ 
                         			$lista_incremental += "," 
                      			} 
                     			 $lista_incremental += "`"" + $($database.StorageGroupname) + "`"" 
 			}
                   } 
 
                } 
             } 
          } 
    } 
 
 
 
 foreach ($pf in Get-PublicFolderDatabase -Server $server -Status) 
    { 
       if ($pf.Mounted -and -not $pf.BackupInProgress) 
       { 
          if ($lista_full.Length -gt 0) 
          { 
             $lista_full += "," 
          } 
          $lista_full += "`"" + $($pf.storagegroupname) + "`"" 
       } 
    } 
 
 $hoy = Get-Date 
 $fecha = $hoy.ToString("yyyyMMdd") 
 $log_incr = "Incr_" + $fecha + ".log" 
 $log_full = "Full_" + $fecha + ".log" 
 Write-Output (Get-Date) | out-file -File $log_incr -Append
 cd "Ruta donde este el tdpexcc"
 . ".\tdpexcc.exe" "backup" $lista_incremental  >> $log_incr
 Write-Output (Get-Date) | out-file -File $log_full -Append
 . ".\tdpexcc.exe" "backup" $lista_full "full" >> $log_full
 $fechac = Get-date
 if ((Test-Path "F:\Logs_BCKP\sg.txt") -ne $true)
 {
 New-Item -path "F:\Logs_BCKP" -name "sg.txt" -type File
 }
 Write "------------------------------------------" >> "F:\Logs_BCKP\sg.txt"
 Write $fechac >> "F:\Logs_BCKP\sg.txt"
 Write "FULL" $lista_full >> "F:\Logs_BCKP\sg.txt"
 Write "INCREMENTAL" $lista_incremental >> "F:\Logs_BCKP\sg.txt"
 Write "------------------------------------------" >> "F:\Logs_BCKP\sg.txt"
 foreach ($oldfile in (Get-ChildItem Incr_*.log,Full_*.log))
 { 
    if ($oldfile.LastWriteTime -le $hoy.AddDays(-60)) 
    { 
       Remove-Item $oldfile 
    } 
 } 
`

