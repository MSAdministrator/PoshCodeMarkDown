---
Author: digitalasylum
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3991
Published Date: 2013-03-01t13
Archived Date: 2016-06-20t19
---

# audit ntfs on shares - 

## Description

audit folders/shares and export to excel

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $Excel = New-Object -Com Excel.Application
 $Excel.visible = $True
 $Excel = $Excel.Workbooks.Add()
 
 $wSheet = $Excel.Worksheets.Item(1)
 $wSheet.Cells.item(1,1) = "Folder Path:" 
 $wSheet.Cells.Item(1,2) = "Users/Groups:"
 $wSheet.Cells.Item(1,3) = "Permissions:"
 $wSheet.Cells.Item(1,4) = "Permissions Inherited:"
 
 $WorkBook = $wSheet.UsedRange
 $WorkBook.Interior.ColorIndex = 8
 $WorkBook.Font.ColorIndex = 11
 $WorkBook.Font.Bold = $True
 
 $dirToAudit = Get-ChildItem -Path "c:\inetpub" -recurse | Where {$_.psIsContainer -eq $true}
 
 $intRow = 1
 foreach ($dir in $dirToAudit)
 {
 	$colACL = Get-Acl -Path $dir.FullName
 
 	foreach ($acl in $colACL)
 		{
 			$intRow++
 			$wSheet.Cells.Item($intRow,1) = $dir.FullName
 			
 				foreach ($accessRight in $acl.Access)
 					{
 						$wSheet.Cells.Item($intRow,2) = "$($AccessRight.IdentityReference)"
 				    	$wSheet.Cells.Item($intRow,3) = "$($AccessRight.FileSystemRights)"
 						$wSheet.Cells.Item($intRow,4) = $acl.AreAccessRulesProtected
 						$intRow++
 					}
 		}
 	
 }
 $WorkBook.EntireColumn.AutoFit()
`

