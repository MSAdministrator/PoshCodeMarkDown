---
Author: arodd
Publisher: 
Copyright: 
Email: 
Version: 11.2.0
Encoding: ascii
License: cc0
PoshCode ID: 2463
Published Date: 2011-01-18t09
Archived Date: 2015-05-08t23
---

# oracle netapp commvault - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 [Reflection.Assembly]::LoadFile("C:\app\oracle\product\11.2.0\client_1\ODP.NET\bin\2.x\Oracle.DataAccess.dll")
 Import-Module DataOntap
 function nas2snapadd
 {param ($volname)
 Connect-NaController -Name nau-nas2.naucom.com | out-null
 Write-Output $volname
 New-NaSnapshot -Name $volname -SnapName "backup_agent" | Out-Null
 }
 
 function nas1snapadd
 {param ($volname)
 Connect-NaController -Name nau-nas1.naucom.com | out-null
 Write-Output $volname
 New-NaSnapshot -Name $volname -SnapName "backup_agent" | Out-Null
 }
 function nas2snaprem
 {param ($volname)
  Connect-NaController -Name nau-nas2.naucom.com | Out-Null
  $snapstodel = Get-NaSnapshot -Name $volname -snapname backup_agent
  if ($snapstodel -ne $null){
  foreach($sn in $snapstodel){
   Write-Output $volname $sn.Name
   Remove-NaSnapshot -Volume $volname -SnapName $sn.Name -Confirm:$false
  }
 }
 }
 
 function nas1snaprem
 {param ($volname)
  Connect-NaController -Name nau-nas1.naucom.com | out-null
  $snapstodel = Get-NaSnapshot -Name $volname -snapname backup_agent
  if ($snapstodel -ne $null){
  foreach($sn in $snapstodel){
   Write-Output $volname $sn.Name
   Remove-NaSnapshot -Volume $volname -SnapName $sn.Name -Confirm:$false
  }
 }
 }
 
 $nas1vols = @("ncpp_data01", "ncpp_data03", "ncpp_redo01", "ncpp_redo03")
 $nas2vols = @("ncpp_data02", "ncpp_data04", "ncpp_data05", "ncpp_redo02", "ncpp_redo04")
 
 foreach($vol in $nas1vols){
 nas1snaprem $vol
 }
 foreach($vol in $nas2vols){
 nas2snaprem $vol
 }
 
 $constr = "User Id=backup_agent;Password=password;Data Source=ncrp"
 $sql="ALTER DATABASE BEGIN BACKUP"
 $sql2="ALTER DATABASE END BACKUP"
 $conn = New-Object Oracle.DataAccess.Client.OracleConnection($constr)
 $conn.Open()
 $command = New-Object Oracle.DataAccess.Client.OracleCommand( $sql,$conn)
 $reader=$command.ExecuteReader()
 $reader.Dispose()
 $command.Dispose()
 
 foreach($vol in $nas1vols){
 nas1snapadd $vol
 }
 foreach($vol in $nas2vols){
 nas2snapadd $vol
 }
 
 $command = New-Object Oracle.DataAccess.Client.OracleCommand( $sql2,$conn)
 $reader=$command.ExecuteReader()
 $reader.Dispose()
 $command.Dispose()
 $conn.Close()
 
 $psi = new-object System.Diagnostics.ProcessStartInfo
 $psi.Verb = "runas"
 $psi.FileName = "N:\scripts\ncpp_nas1.bat"
 [System.Diagnostics.Process]::Start($psi) | out-null
 $psi.FileName = "N:\scripts\ncpp_nas2.bat"
 [System.Diagnostics.Process]::Start($psi) | out-null
`

