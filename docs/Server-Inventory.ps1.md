---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 930
Published Date: 
Archived Date: 2009-03-15t01
---

# server inventory - 

## Description

this script will create an inventory of your server environment.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 
 $ListFile = Read-Host "Please Enter full path to Server list"
 $FileLocation = Read-Host "Please Enter Complete Path and file name to save the output.  Must end in .csv or .txt"
 
 $VCServer = Read-Host "Please Enter Name/IP of Virtual Center Server"
 Connect-VIServer $VCServer
 
 $stats = @()
 
 Get-Content $ListFile | % {
 $OS = get-wmiobject Win32_OperatingSystem -computername $_
 $drive = get-wmiobject win32_logicaldisk -ComputerName $_
 $Computer = Get-QADComputer -Name $_
 $IP = Get-WmiObject -Class Win32_PingStatus -filter "address='$_'"
 $VMSession = Get-VM $_*
 
 $row= "" | Select "Date of Report","Server Name","IP Address","Operating System","Service Pack","Description","OU","ESX Host","RAM (GB)","Drive 1","Drive 1 Description","Drive 1 Free Space","Drive 1 Total Size","Drive 2","Drive 2 Description","Drive 2 Free Space","Drive 2 Total Size","Drive 3","Drive 3 Description","Drive 3 Free Space","Drive 3 Total Size","Drive 4","Drive 4 Description","Drive 4 Free Space","Drive 4 Total Size","Drive 5","Drive 5 Description","Drive 5 Free Space","Drive 5 Total Size","Drive 6","Drive 6 Description","Drive 6 Free Space","Drive 6 Total Size","Drive 7","Drive 7 Description","Drive 7 Free Space","Drive 7 Total Size","Drive 8","Drive 8 Description","Drive 8 Free Space","Drive 8 Total Size"
 
 $row."Date of Report" = Get-date
 $row."Server Name" = $_
 $row."IP Address" = $IP.ProtocolAddress
 $row."Operating System" = $OS.Caption
 $row."Service Pack" = $OS.CSDVersion
 $row."Description" = $Computer.Description
 $row."OU" = $Computer.DN
 $row."ESX Host" = $VMSession.Host.Name
 $row."RAM (GB)" = [math]::Round($VMSession.MemoryMB/1024,3)
 $row."Drive 1" = $drive[0].Caption
 $row."Drive 1 Description" = $drive[0].Description
 $row."Drive 1 Free Space" = [math]::Round($drive[0].Freespace/1024/1024/1024,2)
 $row."Drive 1 Total Size" = [math]::Round($drive[0].Size/1024/1024/1024,2)
 $row."Drive 2" = $drive[1].Caption
 $row."Drive 2 Description" = $drive[1].Description
 $row."Drive 2 Free Space" = [math]::Round($drive[1].Freespace/1024/1024/1024,2)
 $row."Drive 2 Total Size" = [math]::Round($drive[1].Size/1024/1024/1024,2)
 $row."Drive 3" = $drive[2].Caption
 $row."Drive 3 Description" = $drive[2].Description
 $row."Drive 3 Free Space" = [math]::Round($drive[2].Freespace/1024/1024/1024,2)
 $row."Drive 3 Total Size" = [math]::Round($drive[2].Size/1024/1024/1024,2)
 $row."Drive 4" = $drive[3].Caption
 $row."Drive 4 Description" = $drive[3].Description
 $row."Drive 4 Free Space" = [math]::Round($drive[3].Freespace/1024/1024/1024,2)
 $row."Drive 4 Total Size" = [math]::Round($drive[3].Size/1024/1024/1024,2)
 $row."Drive 5" = $drive[4].Caption
 $row."Drive 5 Description" = $drive[4].Description
 $row."Drive 5 Free Space" = [math]::Round($drive[4].Freespace/1024/1024/1024,2)
 $row."Drive 5 Total Size" = [math]::Round($drive[4].Size/1024/1024/1024,2)
 $row."Drive 6" = $drive[5].Caption
 $row."Drive 6 Description" = $drive[5].Description
 $row."Drive 6 Free Space" = [math]::Round($drive[5].Freespace/1024/1024/1024,2)
 $row."Drive 6 Total Size" = [math]::Round($drive[5].Size/1024/1024/1024,2)
 $row."Drive 7" = $drive[6].Caption
 $row."Drive 7 Description" = $drive[6].Description
 $row."Drive 7 Free Space" = [math]::Round($drive[6].Freespace/1024/1024/1024,2)
 $row."Drive 7 Total Size" = [math]::Round($drive[6].Size/1024/1024/1024,2)
 $row."Drive 8" = $drive[7].Caption
 $row."Drive 8 Description" = $drive[7].Description
 $row."Drive 8 Free Space" = [math]::Round($drive[7].Freespace/1024/1024/1024,2)
 $row."Drive 8 Total Size" = [math]::Round($drive[7].Size/1024/1024/1024,2)
 $stats += $row
 }
 
 $stats | Export-Csv $FileLocation -NoTypeInformation
`

