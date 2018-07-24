---
Author: hughs
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3667
Published Date: 2012-09-27t06
Archived Date: 2012-09-29t02
---

# copy files  log to excel - 

## Description

i needed to deploy some dlls to patch a program. this will copy files to a computer and log it in excel. the logging is the only weak link in this script because it uses the credentials of what ever user is running the script. i even included a backout function in the event you need to revert your changes.

## Comments



## Usage



## TODO



## module

`get-md5`

## Code

`#
 #
 $Cred = Get-Credential
 Add-PSSnapin Quest.ActiveRoles.ADManagement -WarningAction SilentlyContinue -ErrorAction SilentlyContinue
 $Computers = Get-QADComputer -SearchRoot "OU=Workstations,DC=Domain,DC=com" -Credential $Cred
 Import-Module BitsTransfer
 
 Function Deploy {
 Clear-Host
 	$Computers | ForEach-Object {
 	$Computer = $_.name
 	$FileToCopy = "C:\Users\Public\File.Here"
 	$Online = Test-Connection -Quiet -ComputerName $Computer
 	IF ($Online -eq $true) {
 		$WMI = Get-WmiObject -Credential $Cred -Class Win32_OperatingSystem -ComputerName $Computer
 		$OSArch = $WMI.OSArchitecture
 		IF ($OSArch -eq '64-bit') {
 			Start-BitsTransfer -Source $FileToCopy -Credential $cred -Description "$Computer - Patch" -Destination "\\$Computer\C$\Program Files (x86)\APPFOLDER\SubFolder\File.Here"
 			}
 		ELSE {
 			Start-BitsTransfer -Source $FileToCopy -Credential $cred -Description "$Computer - Patch" -Destination "\\$Computer\C$\Program Files\APPFOLDER\SubFolder\File.Here"
 			}
 		}
 	}
 }
 
 Taken from: author:fuhj(powershell@live.cn ,http://txj.shell.tor.hu)
 In hind sight I guess I could have used get-sha1 from powershell powerpack.
 #>
 function Get-MD5 ([System.IO.FileInfo] $file = $(throw 'Usage: Get-MD5 [System.IO.FileInfo]')) { 
 	$stream = $null; 
 	$cryptoServiceProvider = [System.Security.Cryptography.MD5CryptoServiceProvider]; 
 	$hashAlgorithm = new-object $cryptoServiceProvider 
 	$stream = $file.OpenRead(); 
 	$hashByteArray = $hashAlgorithm.ComputeHash($stream); 
 	$stream.Close(); 
 	trap {if ($stream -ne $null) {$stream.Close();} 
     break;} 
 	return [string]$hashByteArray; 
 } 
 
 Function GetFileInfo {
 	$Online = Test-Connection -Quiet -ComputerName $Computer
 	IF ($Online -eq $true) {
 		$WMI = Get-WmiObject -Credential $Cred -Class Win32_OperatingSystem -ComputerName $Computer
 		$OSArch = $WMI.OSArchitecture
 		IF ($OSArch -eq '64-bit') {
 			$File = "\\$Computer\C$\Program Files (x86)\APPHERE\SubFolder\File.here"
 			IF ((Test-Path $file) -eq $true) {
 				$MD5 = Get-MD5 $File
 				IF ($MD5 -eq "71 138 251 146 147 253 178 99 136 67 73 107 206 247 60 235"){
 					$c.Cells.Item($intRow,2)  = "PATCHED"}
 				ELSE {$c.Cells.Item($intRow,2)  = "NOT PATCHED"}
 				}
 			ELSE {$c.Cells.Item($intRow,2)  = "FILE MISSING OR ACCESS DENIED"}
 			}
 		ELSE {
 			$File = "\\$Computer\C$\Program Files\APPHERE\SubFolder\File.here"
 			IF ((Test-Path $file) -eq $true) {
 				$MD5 = Get-MD5 $File
 				IF ($MD5 -eq "71 138 251 146 147 253 178 99 136 67 73 107 206 247 60 235"){
 					$c.Cells.Item($intRow,2)  = "PATCHED"}
 				ELSE {$c.Cells.Item($intRow,2)  = "NOT PATCHED"}
 				}
 			ELSE {$c.Cells.Item($intRow,2)  = "FILE MISSING OR ACCESS DENIED"}
 			}	
 		}
 	ELSE {$c.Cells.Item($intRow,2)  = "OFFLINE"}
 }
 
 Function LogIT {
 	$a = New-Object -comobject Excel.Application
 	$a.visible = $True
 	$b = $a.Workbooks.Add()
 	$c = $b.Worksheets.Item(1)
 	$c.Cells.Item(1,1) = "Machine Name"
 	$c.Cells.Item(1,2) = "Version"
 	$c.Cells.Item(1,3) = "Report Time Stamp"
 	$d = $c.UsedRange
 	$d.Interior.ColorIndex = 19
 	$d.Font.ColorIndex = 11
 	$d.Font.Bold = $True
 	$intRow = 2
 	$Computers | ForEach-Object {
 		$Computer = $_.name
 		$c.Cells.Item($intRow,1)  = $Computer
 		GetFileInfo
 		$c.Cells.Item($intRow,3) = Get-date
 		$intRow = $intRow + 1
 		}
 	$d.EntireColumn.AutoFit()
 }
 
 Function DangerWillRobinson {
 	$Computers | ForEach-Object {
 	$Computer = $_.name
 	$Online = Test-Connection -Quiet -ComputerName $Computer
 	IF ($Online -eq $true) {
 		$WMI = Get-WmiObject -Credential $Cred -Class Win32_OperatingSystem -ComputerName $Computer
 		$OSArch = $WMI.OSArchitecture
 		IF ($OSArch -eq '64-bit') {
 			$FILE = Get-WMIObject -computer $Computer -Credential $Cred -query "Select * From CIM_DataFile Where Name ='C:\\Program Files (x86)\\YOUAPP\\APPSSUBFOLDER\\FILE.HERE'"
 			$FILE.Delete()
 			}
 		ELSE {
 			$FILE = Get-WMIObject -computer $Computer -Credential $Cred -query "Select * From CIM_DataFile Where Name ='C:\\Program Files\\YOUAPP\\APPSSUBFOLDER\\FILE.HERE'"
 			$FILE.Delete()
 			}
 		}
 	}
 }
 
 Deploy
 LogIT
`

