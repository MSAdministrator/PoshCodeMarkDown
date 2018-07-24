---
Author: ryan tranchilla
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5754
Published Date: 2015-02-25t09
Archived Date: 2015-10-25t07
---

# get-printerinfo - 

## Description

creates a backup of the configuration of the network printers on a server or computer and exports it to a csv file.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $strComputer = "ComputerName"
 $Ports = get-wmiobject -class "win32_tcpipprinterport" -namespace "root\CIMV2" -computername $strComputer -EnableAllPrivileges
 $Printers = get-wmiobject -class "Win32_Printer" -namespace "root\CIMV2" -computername $strComputer -EnableAllPrivileges
 $ports | Select-Object Name,Hostaddress | Set-Variable port
 $Printers | Select-Object Name,PortName,DriverName,location,Description | Set-Variable print
 $num = 0
 $hash = @{}
 do {
 $hash.Add($port[$num].name, $port[$num].hostaddress)
 $num = $num + 1
 } until ($num -eq $ports.Count)
 
 $table = New-Object system.Data.DataTable �$PrinterInfo�
 $colName = New-Object system.Data.DataColumn PrinterName,([string])
 $colIP = New-Object system.Data.DataColumn IP,([string])
 $colDrive = New-Object system.Data.DataColumn DriverName,([string])
 $colLoc = New-Object system.Data.DataColumn Location,([string])
 $colDesc = New-Object system.Data.DataColumn Description,([string])
 $table.columns.add($colName)
 $table.columns.add($colIP)
 $table.columns.add($colDrive)
 $table.columns.add($colLoc)
 $table.columns.add($colDesc)
 $num = 0
 Do {
 $row = $table.NewRow()
 $row.PrinterName = $printers[$num].name
 $row.IP = $hash.get_item($printers[$num].PortName)
 $row.DriverName = $printers[$num].drivername
 $row.Location = $printers[$num].Location[$num]
 $row.Description = $printers[$num].Description
 $table.Rows.Add($row)
 $num = $num + 1
 } until ($num -eq $printers.Count)
 
 $table | Select-Object PrinterName,IP,DriverName,Location,Description | Export-Csv C:\Printers.csv -NoType
`

