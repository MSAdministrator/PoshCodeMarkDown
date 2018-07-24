---
Author: fred fulford
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3070
Published Date: 2011-11-23t05
Archived Date: 2011-11-29t18
---

# server checks - 

## Description

a script to ping servers (gets the servers from a text file) & report whether they are online or offline, to check free disks space on all servers and to report on any services which are set to automatic startup but are in a stopped state. it puts all this on a nicely formatted excel sheet, then saves a copy as h

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 
 $date = (get-date).tostring("yyyy-MM-dd")
 $filename = "H:\dailychecks\checks_$date.xls"
 
 
 Import-Module "\\emailserver\c$\PS Modules\vamail.videoarts.info.psm1"
 
 
 Start-Process iexplore.exe
 
 $erroractionpreference = "SilentlyContinue"
 $a = New-Object -comobject Excel.Application
 $a.visible = $True 
 
 $b = $a.Workbooks.Add()
 $c = $b.Worksheets.Item(1)
 $d = $b.Worksheets.Item(2)
 $e = $b.Worksheets.Item(3)
 
 $b.name = "$title"
 $c.name = "Stopped Services"
 $d.name = "Free Disk Space"
 $e.name = "Server Connectivity"
 
 
 $c.Cells.Item(1,1) = "STOPPED SERVICES"
 $c.Cells.Item(2,1) = "Machine Name"
 $c.Cells.Item(2,2) = "Service Name"
 $c.Cells.Item(2,3) = "State"
 
 $d.Cells.Item(1,1) = "FREE DISK SPACE"
 $d.Cells.Item(2,1) = "Machine Name"
 $d.Cells.Item(2,2) = "Drive"
 $d.Cells.Item(2,3) = "Total size (MB)"
 $d.Cells.Item(2,4) = "Free Space (MB)"
 $d.Cells.Item(2,5) = "Free Space (%)"
 
 $e.Cells.Item(1,1) = "SERVER CONNECTIVITY"
 $e.Cells.Item(2,1) = "Server Name"
 $e.Cells.Item(2,2) = "Server Status"
 
 
 $c = $c.UsedRange
 $c.Interior.ColorIndex = 19
 $c.Font.ColorIndex = 11
 $c.Font.Bold = $True
 
 $d = $d.UsedRange
 $d.Interior.ColorIndex = 19
 $d.Font.ColorIndex = 11
 $d.Font.Bold = $True
 
 $e = $e.UsedRange
 $e.Interior.ColorIndex = 19
 $e.Font.ColorIndex = 11
 $e.Font.Bold = $True
 $e.EntireColumn.AutoFit()
 
 
 $servRow = 3
 $diskRow = 3
 $pingRow = 3
 
 
 $colservers = Get-Content "C:\dailychecks\Servers.txt"
 foreach ($strServer in $colservers)
 {
 $e.Cells.Item($pingRow, 1) = $strServer.ToUpper()
 
 
 $ping = new-object System.Net.NetworkInformation.Ping
 $Reply = $ping.send($strServer)
 if ($Reply.status �eq �Success�)
 {
 $rightcolor = $e.Cells.Item($pingRow, 2)
 $e.Cells.Item($pingRow, 2) = �Online�
 $rightcolor.interior.colorindex = 10
 }
 else
 {
 
 $wrongcolor = $e.Cells.Item($pingRow, 2)
 $e.Cells.Item($pingRow, 2) = "Offline"
 $wrongcolor.interior.colorindex = 3
 
 }
 $Reply = ""
 
 $pingRow = $pingRow + 1
 
 $e.EntireColumn.AutoFit()
 }
 $colComputers = get-content "C:\dailychecks\Servers.txt"
 foreach ($strComputer in $colComputers)
 {
 $stoppedservices = get-wmiobject Win32_service -computername $strComputer | where{$_.StartMode -eq "Auto" -and $_.State -eq "stopped"} 
 foreach ($objservice in $stoppedservices)
 
 {
  $c.Cells.Item($servRow, 1) = $strComputer.ToUpper()
  $c.Cells.Item($servRow, 2) = $objService.Name
  $c.Cells.Item($servRow, 3) = $objService.State
 $servRow = $servRow + 1
 $c.EntireColumn.AutoFit()
 }
 
 $colDisks = get-wmiobject Win32_LogicalDisk -computername $strComputer -Filter "DriveType = 3" 
 foreach ($objdisk in $colDisks)
 
 {
  $d.Cells.Item($diskRow, 1) = $strComputer.ToUpper()
  $d.Cells.Item($diskRow, 2) = $objDisk.DeviceID
  $d.Cells.Item($diskRow, 3) = "{0:N0}" -f ($objDisk.Size/1024/1024)
  $d.Cells.Item($diskRow, 4) = "{0:N0}" -f ($objDisk.FreeSpace/1024/1024)
  $d.Cells.Item($diskRow, 5) = "{0:P0}" -f ([double]$objDisk.FreeSpace/[double]$objDisk.Size)
 $diskRow = $diskRow + 1
 $d.EntireColumn.AutoFit()
 }
 
 
 }
 
 
 $b.SaveAs($filename, 1)
`

