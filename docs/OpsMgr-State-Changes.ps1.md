---
Author: stefan stranger
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1518
Published Date: 
Archived Date: 2009-12-12t18
---

# opsmgr state changes - 

## Description

state changes for specified opsmgr monitor

## Comments



## Usage



## TODO



## script

``

## Code

`#
 
 
 param ([string]$monitor = $(read-host "Please enter Monitor Name"), [string]$server = $(read-host "Please enter servername hosting OperationsManagerdw database"))
 
 $MonitorDefaultName = "'" + (get-monitor | where {$_.DisplayName -like $monitor}).DisplayName + "'"
 
 if ($MonitorDefaultName -eq "''") {Write-host "No Monitor found"} else 
 {
 
 $sqlConnString = "Server=$server;DataBase=OperationsManagerdw;Integrated Security=SSPI"
 $connection = New-Object System.Data.SqlClient.SqlConnection($sqlConnString)
 $cmd = New-Object System.Data.SqlClient.SqlCommand
 $sqlAdapter = New-Object System.Data.SqlClient.SqlDataAdapter
 $connection.Open() | Out-Null 
 $cmd.CommandTimeout = 30 
 $cmd.Connection = $connection 
 
 $startsql = @'
 SELECT COUNT(*) AS StateChanges, vMonitor.MonitorDefaultName, vManagementPack.ManagementPackDefaultName
 FROM State.vStateRaw INNER JOIN
 vManagedEntityMonitor ON State.vStateRaw.ManagedEntityMonitorRowId = vManagedEntityMonitor.ManagedEntityMonitorRowId INNER JOIN
 vMonitor ON vManagedEntityMonitor.MonitorRowId = vMonitor.MonitorRowId INNER JOIN
 vMonitorManagementPackVersion ON vManagedEntityMonitor.MonitorRowId = vMonitorManagementPackVersion.MonitorRowId INNER JOIN
 vManagementPack ON vMonitor.ManagementPackRowId = vManagementPack.ManagementPackRowId 
 WHERE (MonitorDefaultName = 
 '@
 
 $endsql = @'
 ) 
 AND (vMonitorManagementPackVersion.UnitMonitorInd = 1) 
 GROUP BY vMonitor.MonitorDefaultName, vManagementPack.ManagementPackDefaultName 
 ORDER BY StateChanges DESC
 '@
 
 
 $totalsqlquery = $startsql + $MonitorDefaultName + $endsql
 
 
 $cmd.CommandText = $totalsqlquery
 
 $sqlAdapter = New-Object System.Data.SqlClient.SqlDataAdapter
 $sqlAdapter.SelectCommand = $cmd
 $ds = New-Object System.Data.DataSet
 $sqlAdapter.Fill($ds) | Out-Null
 foreach ($table in $ds.Tables)  
                 {  
                     $table | Format-Table -auto;  
                 }
 $connection.Close()
 }
`

