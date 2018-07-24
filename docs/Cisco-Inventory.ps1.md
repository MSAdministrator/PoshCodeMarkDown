---
Author: kenneth c mazie
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5800
Published Date: 2015-04-01t10
Archived Date: 2015-05-11t20
---

# cisco-inventory.ps1 - 

## Description

as written it will poll cisco routers and switches and if the snmp oid’s match it will pull out model, serial, and ios version.  the resulting spreadsheet contains ip, host name, serial, model, ios version, and rack location.  the script is heavily documented internally.  see the script for more info.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #==================================================================================================
 #
 #
 #=======================================================================================
 
 Clear-Host
 
 #--[ Global presets ]----------------------------------
 $Invocation = (Get-Variable MyInvocation -Scope 0).Value
 #$ScriptPath = Split-Path $Invocation.MyCommand.Path
 $strExe = "f:\usr\bin\snmpget.exe"    #--[ Set the location of the net-snmp tools bin folder ]---------
 $strCommunity = "Public"              #--[ set your community string ]---------
 
 #--[ Assorted Excel presets settings ]----------------------------------
 
 #--[ Create Spreadsheet ]-------------------------------
 $Excel = New-Object -comobject Excel.Application
 $Excel.Visible = $True
 $Excel = $Excel.Workbooks.Add(1)
 $WorkSheet = $Excel.Worksheets.Item(1)
 $WorkSheet.Cells.Item(1,1) = "Target IP"
 $WorkSheet.Cells.Item(1,2) = "Hostname"
 $WorkSheet.Cells.Item(1,3) = "Model #"
 $WorkSheet.Cells.Item(1,4) = "Serial #"
 $WorkSheet.Cells.Item(1,5) = "IOS Ver"
 $WorkSheet.Cells.Item(1,6) = "Location"
 $Workbook = $WorkSheet.UsedRange
 $WorkBook.Interior.ColorIndex = 8
 $WorkBook.Font.ColorIndex = 11
 $WorkBook.Font.Bold = $True
 $WorkBook.EntireColumn.AutoFit()
 
 #--[ Formatting ]----------------------------
 $Col = 1
 while ($Col -le 6){
 	$Edge = 7
 	while ($Edge -le 10){
 		$WorkSheet.Cells.Item(1,$Col).Borders.Item($Edge).LineStyle = 1
 		#$WorkSheet.Cells.Item(1,$Col).Borders.Item($Edge).Weight = 4   #--[ uncomment to make borders bold ]---------
 		$Edge++
 	}
 	$Col++
 }
 
 #$arrDevices = @("192.168.10.2","192.168.10.252")
 $arrDevices = Get-Content ($ScriptPath + "\devicelist.txt")
 
 $intRow = 1
 $count = 0
 
 
 #--[ populate spreadsheet with data ]------------------
 foreach ($strTarget in $arrDevices){ #--[ Cycle through targets ]--------
    $intRow = $intRow + 1 
    $WorkSheet.Cells.Item($intRow,1) = $strTarget #--[ Place Target IP in current row, column A ]----------
    Write-Host "Processing..... " $strTarget
    if (test-connection $strTarget) {
 		if ($count = 5) {$count = 0}
 		
 		$strSerial = iex "cmd.exe /c `"$strExe -v 1 -c $strCommunity $strTarget mib-2.47.1.1.1.1.11.1001`""
 		$strModel = iex "cmd.exe /c `"$strExe -v 1 -c $strCommunity $strTarget mib-2.47.1.1.1.1.13.1001`""
 		$strIOS = iex "cmd.exe /c `"$strExe -v 1 -c $strCommunity $strTarget mib-2.47.1.1.1.1.9.1001`""
 		$strHostName = iex "cmd.exe /c `"$strExe -v 1 -c $strCommunity $strTarget sysName.0`""
 		
 		#--[ If we get back a model place it in current row, column C ]----------
 		if ($strModel.Length -gt 1) {$WorkSheet.Cells.Item($intRow,3) = ($strModel.Split('"'))[1]} 
 		
 		if ($strSerial.Length -gt 1) {$WorkSheet.Cells.Item($intRow,4) = ($strSerial.Split('"'))[1]}
 		
 		#--[ If we get back an IOS version place it in current row, column E ]----------
 		if ($strIOS.Length -gt 1) {$WorkSheet.Cells.Item($intRow,5) = ($strIOS.Split('"'))[1]}
 		
 		#--[ If we get back a hostname place it in current row, column B ]----------
 		if ($strHostname.Length -gt 1) {
 		$strHostName = ($strHostName.Split(' '))[3]
 		$WorkSheet.Cells.Item($intRow,2) = $strHostName 
 		switch($strHostName.substring(1,2)) {
 		   "00" { $errorcode = 'Rack 00' }
 		   "01" { $errorcode = 'Rack 01' }
 		   "02" { $errorcode = 'Rack 02' }
 		   "03" { $errorcode = 'Rack 03' }
 		   "04" { $errorcode = 'Rack 04' }
 		   "05" { $errorcode = 'Rack 05' }
 		   "06" { $errorcode = 'Rack 06' }
 		   "07" { $errorcode = 'Rack 07' }
 		   "08" { $errorcode = 'Rack 08' }
 		   "09" { $errorcode = 'Rack 09' }
 		   "10" { $errorcode = 'Rack 10' }
 		   "11" { $errorcode = 'Rack 11' }
 		   "12" { $errorcode = 'Rack 12' }
 		   "13" { $errorcode = 'Rack 13' }
 		   "14" { $errorcode = 'Rack 14' }
            default { $errorcode = 'Unknown' }
         }  
         $WorkSheet.Cells.Item($intRow,6) = $errorcode #--[ Place location in current row, column F ]----------
 	  }
    } 
 else
 {
 		}
 		
 	$count = $count + 1
 	if ($count =5) {$WorkBook.EntireColumn.AutoFit()}
 }
 
 	$WorkBook.EntireColumn.AutoFit() #--[ Adjust column width across used columns ]---------
 	$WorkSheet.Cells.Item($intRow + 2, 1) = "DONE" 
 	$strFilename = "c:\CiscoInventory-{0:dd-MM-yyyy_HHmm}.xls" -f (Get-Date)  #--[ Places a date stamped spreadsheet in the root of C: ]------
 	$Excel.SaveAs($StrFilename)
 	Write-Host ("Output saved to " + $strFilename)
 	Write-Host "Done..."
`

