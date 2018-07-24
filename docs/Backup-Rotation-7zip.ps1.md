---
Author: steve_o
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3312
Published Date: 2012-04-04t08
Archived Date: 2017-05-22t01
---

# backup rotation (7zip) - 

## Description

this is a utility that uses 3 csv files as drivers to name backups, associate them with files or folders, and to specify a number of versions to retain for each.  backups are deleted on the next run of the utility.  7zip is required.

## Comments



## Usage



## TODO



## script

`create-7zip`

## Code

`#
 #
 First file RotateBackups_MasterList.txt
 RowNbr,BackupName,VersionsRetained,BackupType
 1,TargetBackup,2,Folder
 2,LstDefBackup,5,File
 3,XMLBackup,3,File
 4,SourceBackup,2,Folder
 5,TXTBackup,8,File
 
 Second file RotateBackups_FolderList.txt
 RowNbr,BackupName,FolderName
 1,TargetBackup,c:\MyBooks\target
 2,SourceBackup,c:\MyBooks\source
 
 Third file RotateBackups_FileExtensions.txt
 RowNbr,BackupName,FileExtension,FolderLoc
 1,LstDefBackup,*.def,c:\MyBooks\target
 2,LstDefBackup,*.lst,c:\MyBooks\target
 3,XMLBackup,*.xml,c:\MyBooks\target
 4,TXTBackup,*.txt,c:\MyBooks\rfiles
 
 
 When executed, files are created in c:\Zipfiles that have a name associated with the BackupName, a batch date-time.  BackupName files are counted & compared to VersionsRetained value, and excess ones (oldest first) are marked for deletion upon next run of script.  Can specify a delete to recycle bin (default) or a destructive delete of the backup.  
 
 Use at your own risk.
 
 
 Script:
 
 function create-7zip([String] $aDirectory, [String] $aZipfile){
     [string]$pathToZipExe = "C:\Program Files\7-zip\7z.exe";
     [Array]$arguments = "a", "-tzip", "$aZipfile", "$aDirectory", "-r";
     & $pathToZipExe $arguments;
 }
 
 #************************************************************************************
 #************************************************************************************
 $zipFolder = "C:\ZipFiles"
 $nameConv = "{0:yyyyMMdd_HHmmss}" -f (Get-Date) + ".zip"
 $fileList = @{}
 $FileCountArray = @()
 $bkupTypeArr = @()
 $myDocFolder = "c:\Documents and Settings\MyPC\My Documents\"
 
 $bkupRotateMasterArr = Import-Csv $myDocFolder"RotateBackups_MasterList.txt"
 $fldrBkupArray = Import-Csv $myDocFolder"RotateBackups_FolderList.txt"
 $fileExtBkupArr = Import-Csv $myDocFolder"RotateBackups_FileExtensions.txt"
 
 $KillOrRecycle = "Recycle"
 #************************************************************************************
 #************************************************************************************
 
 $bkup_Counts = @{}
 $b = $null
 foreach($b in $bkupRotateMasterArr)  
     {  
 	$bkup_Counts[$b.BackupName] = $b.VersionsRetained
     }  
 
 
 $type = $null
 foreach ($type in $bkup_Counts.Keys) {
 	$bkupTypeArr += $type
 	}
 
 $type = $null
 $fArray = @{}
 foreach ($type in $bkupRotateMasterArr) {
 	$fArray[$type.BackupName] = ($type.BackupName + $nameConv)
 		}
 		
 if ($fileExtBkupArr) { 		
 	$f = $null 
 	foreach ($f in $fileExtBkupArr) {
 			$arr = @()
 			$arr = (Get-ChildItem $f.FolderLoc -Recurse -Include $f.FileExtension | Select-Object fullname)
 			foreach ($a in $arr) {
 				if ($a) {
 				$fileList[$a] = $f.BackupName
 
 
 	$f = $null
 	foreach ($f in $fileList.Keys) {
 		$arcFile = $null
 		if ($fileList.ContainsKey($f)) {
 			if ($fArray.ContainsKey($fileList[$f])) {
 				$arcFile = $fArray[$fileList[$f]]
 				create-7zip $f.FullName $zipFolder\$arcFile
 
 	$f = $null
 	foreach ($f in $fldrBkupArray) {
 		$arcFldr = $null
 		if ($fArray.ContainsKey($f.BackupName)) {
 			$arcFldr = $fArray[$f.BackupName]
 			create-7zip $f.FolderName $zipFolder\$arcFldr
 
 
 if ($LASTEXITCODE -gt 0)
 	{Throw "7Zip failed" } 
 	Add-Type -AssemblyName Microsoft.VisualBasic
 	$files=get-childitem -path $zipFolder
 	Foreach($file in $files) { 
 		If((Get-ItemProperty -Path $file.fullname).attributes -band [io.fileattributes]::archive)
 		{ Write-output "$file is set to be retained" }
 		ELSE {
 			if ($KillOrRecycle = "Recycle") {
 				Write-output "$file does not have the archive bit set. Deleting (Sent to recycle bin)."
 				[Microsoft.VisualBasic.FileIO.Filesystem]::DeleteFile($file.fullname,'OnlyErrorDialogs','SendToRecycleBin')
 				$output = $_.ErrorDetails
 				}
 			ELSE {
 				Write-output "$file does not have the archive bit set. Deleting."
 				remove-item -recurse $file.fullname
 				$output =$_.ErrorDetails 
 				}
 			} 
 
 	$bkup_counts | Export-Clixml bkup_counts.xml
 
 	$btype = $null
 	foreach ($btype in $bkupTypeArr) {
 		$FileCountArray += ,@(($btype),(dir $zipFolder\$btype"*.zip").count)
 		}
 		
 	$bkup_Counts= Import-Clixml bkup_counts.xml
 
 	attrib $zipFolder"\*" +a 
 	
 	$row = $null
 	foreach ($row in $bkup_Counts.Keys) {
 		Get-ChildItem -Path $zipFolder -Include $row"*" -Recurse #|
 		(dir $zipFolder\$row"*".zip).count - $bkup_Counts[$row]
 		$delfiles = 0
 		$delfiles = (dir $zipFolder\$row"*".zip).count - $bkup_Counts[$row]
 			dir $zipFolder\$row"*" | sort-object -property {$_.CreationTime} |
 			select-object -first $delfiles |
 			foreach-object { attrib $_.FULLNAME -A} 
 		
`

