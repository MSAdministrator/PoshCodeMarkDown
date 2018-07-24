---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5484
Published Date: 2016-10-05t19
Archived Date: 2016-03-05t00
---

# vlfs to excel formatted - 

## Description

this script iterates through a list of servers and writes to excel

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .NOTES 
 Name: Number of VLFs
 Author: Rob Sewell http://sqldbawithabeard.com
 Requires: 
 Version History: 
 .Synopsis
 Number of VLFs and AUtogrowth for every database log file
 .DESCRIPTION
 This script iterates through a list of servers and writes to Excel
 some information about each database log file. It will alter the 
 colour of the cell depending on the paramters you set at the top of the
 script
 .EXAMPLE
 Define the location of the servers on Line 30
 
 $Servers = Get-Content "PATHTOSERVERSFILE.txt"
 
 or (alter locations to fit)
 $Query = "SELECT Name FROM dbo.databases"
 $Servers = Invoke-Sqlcmd -ServerInstance MANAGEMENTSERVER -Database DBADATABASE -Query $query
 
 Define the parameters on line 30 & 31
 .NOTES
 
 This script WILL STOP ALL Excel processes
 #>
 
 $Servers = Get-Content 'PATHTO\sqlservers.txt'
 
 $xl = new-object -comobject excel.application
 
 
 $cells=$ws.Cells
 $Row = 3
 $Col = 2
 $Date = Get-Date
 $Title = 'Results of Script to show VLFs and File Growth run on ' + $Date
 $cells.item(2,2)="Server"
 $cells.item(2,2).font.size=16
 $cells.item(2,3)="Database"
 $cells.item(2,3).font.size=16
 $cells.item(2,4)="No. of VLFs"
 $cells.item(2,4).font.size=16
 $cells.item(2,5)="Growth"
 $cells.item(2,5).font.size=16
 $cells.item(2,6)="Growth Type"
 $cells.item(2,6).font.size=16
 $cells.item(2,7)="Size"
 $cells.item(2,7).font.size=16
 $cells.item(2,8)="Used Space"
 $cells.item(2,8).font.size=16
 $cells.item(2,9)="Name"
 $cells.item(2,9).font.size=16
 $cells.item(2,10)="File Name"
 $cells.item(2,10).font.size=16
 
 foreach ($Server in $Servers)
 {  
 	$srv = New-Object Microsoft.SqlServer.Management.Smo.Server $Server
 	foreach ($db in $srv.Databases|Where-Object {$_.isAccessible -eq $True})
 	{
 		$Col = 2
 		$Row++
 		$VLF = $DB.ExecuteWithResults("DBCC LOGINFO").Tables[0].Rows.Count
 		$logFile = $db.LogFiles | Select Growth,GrowthType,Size, UsedSpace,Name,FileName
 		$Name = $DB.name
 		$cells.item($row,$col)=$Server
 		$col++
 		$cells.item($row,$col)=$Name
 		$col++
 		if($VLF -gt $TooMany) 
 		{
 		}
 		if($VLF -gt $WayTooMany)
 		{
 		}
 		$cells.item($row,$col)=$VLF
 		$col++
 		$cells.item($row,$col)=$logFile.Growth
 		$col++
 		$Type = $logFile.GrowthType.ToString()
 		if($Type -eq 'Percent')
 		{
 		}
 		$cells.item($row,$col)=$Type
 		$col++
 		$cells.item($row,$col)=($logFile.Size)
 		$col++
 		$cells.item($row,$col)=($logFile.UsedSpace)
 		$col++
 		$cells.item($row,$col)=$logFile.Name
 		$col++
 		$cells.item($row,$col)=$logFile.FileName
 	}
 	$Row++
 }
 $ws.UsedRange.EntireColumn.AutoFit()
 $cells.item(1,2)=$Title 
 $cells.item(1,2).font.size=24
 $cells.item(1,2).font.bold=$True
 $cells.item(1,2).font.underline=$True
 $Date = Get-Date -f ddMMyy
 $filename = 'VLF' + $Date
 $wb.Saveas("C:\temp\$filename.xlsx")
 $xl.quit()
 Stop-Process -Name EXCEL
`

