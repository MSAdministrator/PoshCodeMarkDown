---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jan.egil.ring@powershell.no
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6895
Published Date: 2017-05-17t23
Archived Date: 2017-05-21t01
---

# create-printers.ps1 - create-printers.ps1

## Description

simple script to bulk-create printers on a print-server. printers are imported from a csv-file.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 
 
 function CreatePrinter {
 $server = $args[0]
 $print = ([WMICLASS]"\\$server\ROOT\cimv2:Win32_Printer").createInstance() 
 $print.drivername = $args[1]
 $print.PortName = $args[2]
 $print.Shared = $true
 $print.Sharename = $args[3]
 $print.Location = $args[4]
 $print.Comment = $args[5]
 $print.DeviceID = $args[6]
 $print.Put() 
 }
 
 function CreatePrinterPort {
 $server =  $args[0] 
 $port = ([WMICLASS]"\\$server\ROOT\cimv2:Win32_TCPIPPrinterPort").createInstance() 
 $port.Name= $args[1]
 $port.SNMPEnabled=$false 
 $port.Protocol=1 
 $port.HostAddress= $args[2] 
 $port.Put() 
 }
 
 $printers = Import-Csv c:\printers.csv
 
 foreach ($printer in $printers) {
 CreatePrinterPort $printer.Printserver $printer.Portname $printer.IPAddress
 CreatePrinter $printer.Printserver $printer.Driver $printer.Portname $printer.Sharename $printer.Location $printer.Comment $printer.Printername
 }
`

