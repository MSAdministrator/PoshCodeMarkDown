---
Author: pedro genil
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3908
Published Date: 2013-01-18t09
Archived Date: 2013-01-22t15
---

# backup full - 

## Description

en exchange 2007, una vez balanceado el nodo. es neceario realizar un backup full sobre cada sg.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 If ((Get-PSSnapin | where {$_.Name -match "Exchange.Management"}) -eq $null)
 {
 	Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Admin
 }
 $lista= ""
 [int]$tam=30
 $server = "MAILBOX"
 $discos=get-wmiobject -class win32_volume -filter "DriveType=3" -computer $server
 $databases = Get-MailboxDatabase -Server $server -Status
 foreach ($database in $databases)
 {
          if (-not $database.Recovery) 
          { 
             if ($database.Mounted) 
             { 
                if (-not $database.BackupInProgress) 
                { 
                     $logpath = (get-storagegroup $database.storagegroup).logfolderpath
                     foreach ($disco in $discos)
                         {
                              if ($disco.driveletter -eq $logpath.drivename)
                                 {
                                     $tamlog= ($disco.freespace/1GB)
                                         if ($tamlog -lt $tam)
                                                 {
                                                     if ($lista.Length -gt 0) 
                      			                            { 
                         			                             $lista += "," 
                      			                            } 
                     			                    $lista += "`"" + $($database.StorageGroupname) + "`"" 
                                                     $tamlog = $tam
                                                  }
 
                                 }   
                         }
                 }
            }
          }
 }
 $hoy = Get-Date 
 $fecha = $hoy.ToString("yyyyMMdd") 
 $log_full = "Full_" + $fecha + ".log" 
 cd "Ruta ejecutable del tdpexcc"
 . ".\tdpexcc.exe" "backup" $lista "full" >> $log_full
 $fechac = Get-date
 if ((Test-Path "F:\Logs_BCKP\full.txt") -ne $true)
 {
 New-Item -path "F:\Logs_BCKP" -name "full.txt" -type File
 }
 Write "------------------------------------------" >> "F:\Logs_BCKP\full.txt"
 Write $fechac >> "F:\Logs_BCKP\full.txt"
 Write "FULL" $lista_full >> "F:\Logs_BCKP\full.txt"
 Write "------------------------------------------" >> "F:\Logs_BCKP\full.txt"
 foreach ($oldfile in (Get-ChildItem Full_*.log))
 { 
    if ($oldfile.LastWriteTime -le $hoy.AddDays(-60)) 
    { 
       Remove-Item $oldfile 
    } 
 }
`

