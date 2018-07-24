---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 3.1.19
Encoding: ascii
License: cc0
PoshCode ID: 4508
Published Date: 2013-10-07t21
Archived Date: 2013-10-11t21
---

# createmasterlists - 

## Description

powershell form that can create master lists for later use in quests notes migration tool…

## Comments



## Usage



## TODO



## function

`call-createmasterlists_pff`

## Code

`#
 #
 #========================================================================
 #========================================================================
 #----------------------------------------------
 #----------------------------------------------
 
 function OnApplicationLoad {
 	
 }
 
 function OnApplicationExit {
 	
 }
 
 
 #----------------------------------------------
 #----------------------------------------------
 function Call-CreateMasterLists_pff {
 
 	#----------------------------------------------
 	#----------------------------------------------
 	[void][reflection.assembly]::Load("mscorlib, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.Windows.Forms, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.Data, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.Drawing, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")
 	[void][reflection.assembly]::Load("System.Xml, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.DirectoryServices, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")
 	[void][reflection.assembly]::Load("System.Core, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 	[void][reflection.assembly]::Load("System.ServiceProcess, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")
 
 	#----------------------------------------------
 	#----------------------------------------------
 	[System.Windows.Forms.Application]::EnableVisualStyles()
 	$formCreateNMEMasterList = New-Object 'System.Windows.Forms.Form'
 	$checkboxUseWildcardSearchkey = New-Object 'System.Windows.Forms.CheckBox'
 	$labelNumberOfFilesToSplit = New-Object 'System.Windows.Forms.Label'
 	$ColumnsForKey = New-Object 'System.Windows.Forms.ComboBox'
 	$labelColumnToUseAsKeyforV = New-Object 'System.Windows.Forms.Label'
 	$FileBuildingProgress = New-Object 'System.Windows.Forms.ProgressBar'
 	$statusbar = New-Object 'System.Windows.Forms.StatusBar'
 	$buttonBuildFiles = New-Object 'System.Windows.Forms.Button'
 	$buttonBrowseFolder = New-Object 'System.Windows.Forms.Button'
 	$textboxFolder = New-Object 'System.Windows.Forms.TextBox'
 	$NumberOfFiles = New-Object 'System.Windows.Forms.ComboBox'
 	$checkboxSplitTheFilesForMe = New-Object 'System.Windows.Forms.CheckBox'
 	$labelSaveFilesTo = New-Object 'System.Windows.Forms.Label'
 	$checkboxMigrateAllUsersusedT = New-Object 'System.Windows.Forms.CheckBox'
 	$buttonUserMigrateBrowse = New-Object 'System.Windows.Forms.Button'
 	$textboxUserToMigrate = New-Object 'System.Windows.Forms.TextBox'
 	$labelUsersToMigratevalues = New-Object 'System.Windows.Forms.Label'
 	$buttonMasterListBrowse = New-Object 'System.Windows.Forms.Button'
 	$textboxMasterTSV = New-Object 'System.Windows.Forms.TextBox'
 	$labelSourceMasterTsv = New-Object 'System.Windows.Forms.Label'
 	$openfiledialog1 = New-Object 'System.Windows.Forms.OpenFileDialog'
 	$openfiledialog2 = New-Object 'System.Windows.Forms.OpenFileDialog'
 	$folderbrowserdialog1 = New-Object 'System.Windows.Forms.FolderBrowserDialog'
 	$folderbrowserdialog2 = New-Object 'System.Windows.Forms.FolderBrowserDialog'
 	$InitialFormWindowState = New-Object 'System.Windows.Forms.FormWindowState'
 
 	#----------------------------------------------
 	#----------------------------------------------
 	
 	
 	
 	
 	
 	
 	
 	$formCreateNMEMasterList_Load={
 	
 	}
 	
 	$buttonMasterListBrowse_Click={
 	
 		if($openfiledialog1.ShowDialog() -eq 'OK')
 		{
 			$textboxMasterTSV.Text = $openfiledialog1.FileName
 		}
 	}
 	
 	$buttonUserMigrateBrowse_Click={
 	
 		if($openfiledialog2.ShowDialog() -eq 'OK')
 		{
 			$textboxUserToMigrate.Text = $openfiledialog2.FileName
 		}
 	}
 	
 	$buttonBrowseFolder_Click={
 		if($folderbrowserdialog1.ShowDialog() -eq 'OK')
 		{
 			$textboxFolder.Text = $folderbrowserdialog1.SelectedPath
 		}
 	}
 	
 	function Load-ComboBox 
 	{
 	<#
 		.SYNOPSIS
 			This functions helps you load items into a ComboBox.
 	
 		.DESCRIPTION
 			Use this function to dynamically load items into the ComboBox control.
 	
 		.PARAMETER  ComboBox
 			The ComboBox control you want to add items to.
 	
 		.PARAMETER  Items
 			The object or objects you wish to load into the ComboBox's Items collection.
 	
 		.PARAMETER  DisplayMember
 			Indicates the property to display for the items in this control.
 		
 		.PARAMETER  Append
 			Adds the item(s) to the ComboBox without clearing the Items collection.
 		
 		.EXAMPLE
 			Load-ComboBox $combobox1 "Red", "White", "Blue"
 		
 		.EXAMPLE
 			Load-ComboBox $combobox1 "Red" -Append
 			Load-ComboBox $combobox1 "White" -Append
 			Load-ComboBox $combobox1 "Blue" -Append
 		
 		.EXAMPLE
 			Load-ComboBox $combobox1 (Get-Process) "ProcessName"
 	#>
 		Param (
 			[ValidateNotNull()]
 			[Parameter(Mandatory=$true)]
 			[System.Windows.Forms.ComboBox]$ComboBox,
 			[ValidateNotNull()]
 			[Parameter(Mandatory=$true)]
 			$Items,
 		    [Parameter(Mandatory=$false)]
 			[string]$DisplayMember,
 			[switch]$Append
 		)
 		
 		if(-not $Append)
 		{
 			$ComboBox.Items.Clear()	
 		}
 		
 		if($Items -is [Object[]])
 		{
 			$ComboBox.Items.AddRange($Items)
 		}
 		elseif ($Items -is [Array])
 		{
 			$ComboBox.BeginUpdate()
 			foreach($obj in $Items)
 			{
 				$ComboBox.Items.Add($obj)	
 			}
 			$ComboBox.EndUpdate()
 		}
 		else
 		{
 			$ComboBox.Items.Add($Items)	
 		}
 	
 		$ComboBox.DisplayMember = $DisplayMember	
 	}
 	
 	function Load-ListBox 
 	{
 	<#
 		.SYNOPSIS
 			This functions helps you load items into a ListBox or CheckedListBox.
 	
 		.DESCRIPTION
 			Use this function to dynamically load items into the ListBox control.
 	
 		.PARAMETER  ListBox
 			The ListBox control you want to add items to.
 	
 		.PARAMETER  Items
 			The object or objects you wish to load into the ListBox's Items collection.
 	
 		.PARAMETER  DisplayMember
 			Indicates the property to display for the items in this control.
 		
 		.PARAMETER  Append
 			Adds the item(s) to the ListBox without clearing the Items collection.
 		
 		.EXAMPLE
 			Load-ListBox $ListBox1 "Red", "White", "Blue"
 		
 		.EXAMPLE
 			Load-ListBox $listBox1 "Red" -Append
 			Load-ListBox $listBox1 "White" -Append
 			Load-ListBox $listBox1 "Blue" -Append
 		
 		.EXAMPLE
 			Load-ListBox $listBox1 (Get-Process) "ProcessName"
 	#>
 		Param (
 			[ValidateNotNull()]
 			[Parameter(Mandatory=$true)]
 			[System.Windows.Forms.ListBox]$ListBox,
 			[ValidateNotNull()]
 			[Parameter(Mandatory=$true)]
 			$Items,
 		    [Parameter(Mandatory=$false)]
 			[string]$DisplayMember,
 			[switch]$Append
 		)
 		
 		if(-not $Append)
 		{
 			$listBox.Items.Clear()	
 		}
 		
 		if($Items -is [System.Windows.Forms.ListBox+ObjectCollection])
 		{
 			$listBox.Items.AddRange($Items)
 		}
 		elseif ($Items -is [Array])
 		{
 			$listBox.BeginUpdate()
 			foreach($obj in $Items)
 			{
 				$listBox.Items.Add($obj)
 			}
 			$listBox.EndUpdate()
 		}
 		else
 		{
 			$listBox.Items.Add($Items)	
 		}
 	
 		$listBox.DisplayMember = $DisplayMember	
 	}
 	
 	$buttonBrowseFolder_Click2={
 		if($folderbrowserdialog2.ShowDialog() -eq 'OK')
 		{
 			$textboxFolder.Text = $folderbrowserdialog2.SelectedPath
 		}
 	}
 	
 	$checkboxMigrateAllUsersusedT_CheckedChanged={
 		if ($checkboxMigrateAllUsersusedT.Checked) {
 			
 			$PathToMasterFile = $textboxMasterTSV.Text
 			
 			if ($(Test-Path $PathToMasterFile)) {
 				$checkboxSplitTheFilesForMe.Enabled = $true
 				$checkboxSplitTheFilesForMe.Checked = $true
 				$UsersToMigrate = Get-Content $PathToMasterFile
 				if ($UsersToMigrate.count -gt 1000 -and $checkboxMigrateAllUsersusedT.Checked -eq $true) {
 					Load-ComboBox $NumberOfFiles "2"
 					3..500 | % { Load-ComboBox $NumberOfFiles "$_" -Append }
 				}
 				else {
 					Load-ComboBox $NumberOfFiles "2"
 					3..$($UsersToMigrate.count/2) | % { Load-ComboBox $NumberOfFiles "$_" -Append }
 				}
 				$NumberOfFiles.Enabled = $true
 				$NumberOfFiles.Focus
 			}
 			
 			$textboxUserToMigrate.Enabled = $false
 			$buttonUserMigrateBrowse.Enabled = $false
 			$ColumnsForKey.Enabled = $false
 			$checkboxUseWildcardSearchkey.Enabled = $false
 		}
 		else {
 			$textboxUserToMigrate.Enabled = $true
 			$buttonUserMigrateBrowse.Enabled = $true
 			$ColumnsForKey.Enabled = $true
 			$checkboxUseWildcardSearchkey.Enabled = $true
 		}
 	}
 	
 	function DisableFormInput
 	{
 		$textboxMasterTSV.Enabled = $false
 		$buttonMasterListBrowse.Enabled = $false
 		$textboxUserToMigrate.Enabled = $false
 		$buttonUserMigrateBrowse.Enabled = $false
 		$buttonBrowseFolder.Enabled = $false
 		$textboxFolder.Enabled = $false
 		$ColumnsForKey.Enabled = $false
 		$NumberOfFiles.Enabled = $false
 		$checkboxSplitTheFilesForMe.Enabled = $false
 		$textboxUserToMigrate.Enabled = $false
 		$buttonUserMigrateBrowse.Enabled = $false
 		$ColumnsForKey.Enabled = $false
 		$buttonBuildFiles.Enabled = $false
 		$checkboxMigrateAllUsersusedT.Enabled = $false
 		$checkboxUseWildcardSearchkey.Enabled = $false
 	}
 	
 	function EnableFormInput
 	{
 		$textboxMasterTSV.Enabled = $true
 		$buttonMasterListBrowse.Enabled = $true
 		$buttonBrowseFolder.Enabled = $true
 		$textboxFolder.Enabled = $true
 		$checkboxSplitTheFilesForMe.Enabled = $true
 		$buttonBuildFiles.Enabled = $true
 		$checkboxMigrateAllUsersusedT.Enabled = $true
 		
 		if ($checkboxSplitTheFilesForMe.Checked) {
 			$NumberOfFiles.Enabled = $true
 		}
 		else {
 			$NumberOfFiles.Enabled = $false
 		}
 		
 		if ($checkboxMigrateAllUsersusedT.Checked) {
 			$textboxUserToMigrate.Enabled = $false
 			$buttonUserMigrateBrowse.Enabled = $false
 			$ColumnsForKey.Enabled = $false
 			$checkboxUseWildcardSearchkey.Enabled = $false
 		}
 		else {
 			$textboxUserToMigrate.Enabled = $true
 			$buttonUserMigrateBrowse.Enabled = $true
 			$ColumnsForKey.Enabled = $true
 			$checkboxUseWildcardSearchkey.Enabled = $true
 		}
 	}
 	
 	$buttonBuildFiles_Click={
 		
 		DisableFormInput
 		
 		$FileBuildingProgress.Value = 0
 		$statusbar.Text = "Starting..."
 		
 		$PathToMasterFile = $textboxMasterTSV.Text
 		$PathToUserToMigrateFile = $textboxUserToMigrate.Text
 		$PathForNewFiles = $textboxFolder.Text
 	
 		$MasterFileOK = Test-Path $PathToMasterFile
 		
 		if ($checkboxMigrateAllUsersusedT.Checked -eq $true) {
 			$UsersToMigrateFileOK = $true
 		}
 		else {
 			$UsersToMigrateFileOK = Test-Path $PathToUserToMigrateFile
 		}
 			
 		$SaveFolderOK = Test-Path $PathForNewFiles
 		
 		$ColumnKey = $ColumnsForKey.Text
 		$MasterFileBN = Get-ChildItem $PathToMasterFile | Select-Object -ExpandProperty BaseName
 		$TmpCsvFile = "$PathForNewFiles\$MasterFileBN-TEMP.tmp"
 		$ErrorFile = "$PathForNewFiles\NotFound.txt"
 		$NumberFilesWanted=$NumberOfFiles.Text
 		
 		Remove-Item $ErrorFile -Force -ErrorAction 'SilentlyContinue'
 		
 		if ($ColumnKey -ne "" -and $ColumnKey -ne $null) {
 			$ColumnKeyOK = $true
 		}
 		elseif ($checkboxMigrateAllUsersusedT.Checked -eq $true) {
 			$ColumnKeyOK = $true
 		}
 		else {
 			$ColumnKeyOK = $false	
 		}
 		
 		if ($MasterFileOK -eq $true -and $UsersToMigrateFileOK -eq $true -and $SaveFolderOK -eq $true -and $ColumnKeyOK -eq $true) {
 			
 			if ($checkboxMigrateAllUsersusedT.Checked -eq $true) {
 				$UsersToSplit = Get-Content $PathToMasterFile -Encoding 'Unicode'
 				$NumberOfUsersToMigrate=$UsersToSplit.count
 			}
 			else {
 				$statusbar.Text = "Loading files..."
 				$MasterList = Import-csv $PathToMasterFile -Delimiter "`t" -Encoding 'Unicode'
 				$UsersToMigrate = Get-Content $PathToUserToMigrateFile
 				
 				$NumberOfUsersToMigrate=$UsersToMigrate.count
 				$FindingUserNr=0
 				foreach ($UserToMigrate in $UsersToMigrate) {
 					$FindingUserNr++
 					$FileBuildingProgress.Value = $FindingUserNr/$NumberOfUsersToMigrate*100
 					$statusbar.Text = "Finding $UserToMigrate in master file..."
 					[System.Windows.Forms.Application]::DoEvents()
 					if ($checkboxUseWildcardSearchkey.Checked -eq $true) {
 						$Match = $null
 						$Match = $MasterList | Where-Object { $_.$($ColumnKey) -like "*$UserToMigrate*" }
 						if ($Match -ne $null) {
 							$Match | Export-Csv $TmpCsvFile -Delimiter "`t" -Encoding 'Unicode' -Append -NoTypeInformation
 						}
 						else {
 							Write-Output "No match for $UserToMigrate (wildcard)" | Out-File $ErrorFile -Append
 						}
 					}
 					else {
 						$Match = $null
 						$Match = $MasterList | Where-Object { $_.$($ColumnKey) -eq $UserToMigrate }
 						if ($Match -ne $null) {
 							$Match | Export-Csv $TmpCsvFile -Delimiter "`t" -Encoding 'Unicode' -Append -NoTypeInformation
 						}
 						else {
 							Write-Output "No match for $UserToMigrate (no wildcard)" | Out-File $ErrorFile -Append
 						}
 					}
 				}
 				
 				$statusbar.Text = "Reading temp file..."
 				$UsersToSplit = Get-Content $TmpCsvFile -Encoding 'Unicode' | % { $_ -replace "`"" }
 			Remove-Item $TmpCsvFile -Force -ErrorAction 'SilentlyContinue'
 		}
 
 		$statusbar.Text = "Preparing to create final file(s)... Wait..."
 		$FileBuildingProgress.Value = 0
 		[int] $NumberOfUsersPerFile=$NumberOfUsersToMigrate/$NumberFilesWanted
 		[int] $CurrentFileNumber = 1
 		[int] $CurrentUser = 0
 		[int] $CurrentUserNoReset = 0
 		$FirstRun = $true
 		$Headers = Get-Content $PathToMasterFile -Encoding 'Unicode' | select -First 1
 		
 		Remove-Item $("$PathForNewFiles\$MasterFileBN-ChosenUsers.tsv") -Force -ErrorAction 'SilentlyContinue'
 		Remove-Item $("$PathForNewFiles\$MasterFileBN-1.tsv") -Force -ErrorAction 'SilentlyContinue'
 		
 		if ($checkboxSplitTheFilesForMe.Checked) {
 			$UsersToSplit | % {
 				$CurrentUserNoReset++
 				$statusbar.Text = "Splitting into files..."
 				$FileBuildingProgress.Value = $CurrentUserNoReset/$NumberOfUsersToMigrate*100
 				
 				[System.Windows.Forms.Application]::DoEvents()
 				$CurrentUser++
 				
 				if ($CurrentUser -ge $NumberOfUsersPerFile -and $CurrentFileNumber -lt $NumberFilesWanted -and $FirstRun -eq $false) {
 					$CurrentFileNumber++
 					[int] $CurrentUser = 0
 					$Headers | Out-File $("$PathForNewFiles\$MasterFileBN-$CurrentFileNumber.tsv") -Encoding 'Unicode'
 				}
 
 				$_ | Out-File $("$PathForNewFiles\$MasterFileBN-$CurrentFileNumber.tsv") -Encoding 'Unicode' -Append
 				
 				if ($FirstRun -eq $true -and $CurrentUser -ge 3) {
 					$FirstRun = $false
 				}
 			}
 		}
 		else {
 			$statusbar.Text = "Creating file..."
 			$UsersToSplit | Out-File $("$PathForNewFiles\$MasterFileBN-ChosenUsers.tsv") -Encoding 'Unicode' -Append
 		}
 		$statusbar.Text = "Ready."
 	}
 	else {
 		$statusbar.Text = "Check all the paths/settings."
 	}
 	
 	EnableFormInput
 	$FileBuildingProgress.Value = 0
 }
 
 $textboxMasterTSV_TextChanged={
 	$PathToMasterFile = $textboxMasterTSV.Text
 	if ($(Test-Path $PathToMasterFile)) {
 		$MasterList = Import-csv $PathToMasterFile -Delimiter "`t" -Encoding Unicode | select -First 1
 		$Columns=$MasterList | Get-Member | Where-Object { $_.MemberType -eq 'NoteProperty' } | select -ExpandProperty Name
 		$Columns | Sort-Object | % { Load-ComboBox $ColumnsForKey "$_" -Append }
 	}
 	
 }
 
 $textboxFolder_TextChanged={
 
 }
 
 $textboxUserToMigrate_TextChanged={
 	$PathToUserToMigrateFile = $textboxUserToMigrate.Text
 	
 	if ($(Test-Path $PathToUserToMigrateFile)) {
 		$checkboxSplitTheFilesForMe.Enabled = $true
 		$UsersToMigrate = Get-Content $PathToUserToMigrateFile
 		Load-ComboBox $NumberOfFiles "2"
 		3..$($UsersToMigrate.count/2) | % { Load-ComboBox $NumberOfFiles "$_" -Append }
 	}
 	else {
 		$NumberOfFiles.Enabled = $false
 		$checkboxSplitTheFilesForMe.Enabled = $false
 	}
 }
 
 $checkboxSplitTheFilesForMe_CheckedChanged={
 	if ($checkboxSplitTheFilesForMe.Checked) {
 		$NumberOfFiles.Enabled = $true
 	}
 	else {
 		$NumberOfFiles.Enabled = $false
 	}
 }
 	#----------------------------------------------
 	#----------------------------------------------
 	
 	$Form_StateCorrection_Load=
 	{
 		$formCreateNMEMasterList.WindowState = $InitialFormWindowState
 	}
 	
 	$Form_Cleanup_FormClosed=
 	{
 		try
 		{
 			$buttonBuildFiles.remove_Click($buttonBuildFiles_Click)
 			$buttonBrowseFolder.remove_Click($buttonBrowseFolder_Click2)
 			$textboxFolder.remove_TextChanged($textboxFolder_TextChanged)
 			$checkboxSplitTheFilesForMe.remove_CheckedChanged($checkboxSplitTheFilesForMe_CheckedChanged)
 			$checkboxMigrateAllUsersusedT.remove_CheckedChanged($checkboxMigrateAllUsersusedT_CheckedChanged)
 			$buttonUserMigrateBrowse.remove_Click($buttonUserMigrateBrowse_Click)
 			$textboxUserToMigrate.remove_TextChanged($textboxUserToMigrate_TextChanged)
 			$buttonMasterListBrowse.remove_Click($buttonMasterListBrowse_Click)
 			$textboxMasterTSV.remove_TextChanged($textboxMasterTSV_TextChanged)
 			$formCreateNMEMasterList.remove_Load($formCreateNMEMasterList_Load)
 			$formCreateNMEMasterList.remove_Load($Form_StateCorrection_Load)
 			$formCreateNMEMasterList.remove_FormClosed($Form_Cleanup_FormClosed)
 		}
 		catch [Exception]
 		{ }
 	}
 
 	#----------------------------------------------
 	#----------------------------------------------
 	#
 	#
 	$formCreateNMEMasterList.Controls.Add($checkboxUseWildcardSearchkey)
 	$formCreateNMEMasterList.Controls.Add($labelNumberOfFilesToSplit)
 	$formCreateNMEMasterList.Controls.Add($ColumnsForKey)
 	$formCreateNMEMasterList.Controls.Add($labelColumnToUseAsKeyforV)
 	$formCreateNMEMasterList.Controls.Add($FileBuildingProgress)
 	$formCreateNMEMasterList.Controls.Add($statusbar)
 	$formCreateNMEMasterList.Controls.Add($buttonBuildFiles)
 	$formCreateNMEMasterList.Controls.Add($buttonBrowseFolder)
 	$formCreateNMEMasterList.Controls.Add($textboxFolder)
 	$formCreateNMEMasterList.Controls.Add($NumberOfFiles)
 	$formCreateNMEMasterList.Controls.Add($checkboxSplitTheFilesForMe)
 	$formCreateNMEMasterList.Controls.Add($labelSaveFilesTo)
 	$formCreateNMEMasterList.Controls.Add($checkboxMigrateAllUsersusedT)
 	$formCreateNMEMasterList.Controls.Add($buttonUserMigrateBrowse)
 	$formCreateNMEMasterList.Controls.Add($textboxUserToMigrate)
 	$formCreateNMEMasterList.Controls.Add($labelUsersToMigratevalues)
 	$formCreateNMEMasterList.Controls.Add($buttonMasterListBrowse)
 	$formCreateNMEMasterList.Controls.Add($textboxMasterTSV)
 	$formCreateNMEMasterList.Controls.Add($labelSourceMasterTsv)
 	$formCreateNMEMasterList.ClientSize = '289, 473'
 	$formCreateNMEMasterList.FormBorderStyle = 'FixedToolWindow'
 	$formCreateNMEMasterList.Icon = [System.Convert]::FromBase64String('
 AAABAAEAMDAAAAAAAACoJQAAFgAAACgAAAAwAAAAYAAAAAEAIAAAAAAAACQAAAAAAAAAAAAAAAAA
 AAAAAAD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A
 LDA1ASwwNQIsMDUFLDA1CCwwNQksMDUILDA1BiwwNQQsMDUCLDA1Af///wD///8A////AP///wD/
 //8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP//
 /wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////
 AAAAAAEAAAABAAAABAAAAAYHCAkKDg8RDxUXGRUaGx8aGx4hHR0gIx4fISUbICMmFiEkKBEiJCkJ
 IyYqAwAAAAEAAAABAAAAAQAAAAEAAAABAAAAAQAAAAEAAAABAAAAAQAAAAEAAAABAAAAAQAAAAEA
 AAABAAAAAQAAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAD///8A////AP//
 /wD///8A////AP///wD///8AAAAAAQAAAAYAAAAOAAAAGgICAikDAwQ4BgcHSAgJClMKCwxaCwwN
 WAwND1EPERJIEhUXPRYZGzAYGh4eFxgbEg4PEgoDBAQHAAAABgAAAAYAAAAGAAAABgAAAAcAAAAH
 AAAACQAAAAsAAAANAAAADgAAAA4AAAAOAAAADgAAAAwAAAAKAAAABwAAAAQAAAACAAAAAQAAAAEA
 AAAAAAAAAAAAAAD///8A////AP///wD///8A////AP///wAAAAABAAAAAwAAAA0BAQEbGxQIOjon
 C6RKLwf1WzgE/3BFBP96Twj/c04K5kQwDKYICQplDA0PWRETFE4TFRdBEhQVMAwNDiIEBQUbAAAA
 GQAAABkAAAAZAAAAGgAAAB0AAAAkAAAALgAAADkAAABDAAAASwAAAFAAAABQAAAASwAAAEMAAAA3
 AAAAKQAAABsAAAAQAAAACAAAAAQAAAABAAAAAQAAAAD///8A////AP///wD///8AAAAAAQAAAAEA
 AAAEAAAACAAAABIaFQsuMiMN2SgXAP9UNQH/d0sB/4xZAv+cYAH/sXAB/690Bv91UAvQFxIKbwwN
 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
 /wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////ACUlJDAzJxL/OyYB
 /2E/Af91TQH/glYC/4ZaB/+Oayr/vKJy/6pxCP/cmwT/6bQG/4hhA/+unX7/3t7e/9zc3f/n5+b/
 5eXj/5l4O//PhwL/yYQC/3FKAf+CVQH/cUoC/2ZGC/92UAr/o2wF/8OAAf/OhgH/248C/9GIAv+0
 dgL/YEsm/1JLPv8sMDUKLDA1A////wD///8A////AP///wD///8A////AP///wD///8A////AP//
 /wD///8A////AP///wAlJSRgSzQN/1o5AP91TAD/kF8B/6RsAv+0dwL/wX4C/8+MA//fowX/6rUG
 /4lgA/+wn4D/5OPk/+Hh4f/c3Nv/c2FA/8+IBP/WjAL/0ogC/6dtAv+DVQL/oGgB/7Z2Av/EgAH/
 0YgC/9+RA//hlAP/3pEC/759Bf+PeVL/npyY/1dQQ/8sMDUELDA1Af///wD///8A////AP///wD/
 //8A////AP///wD///8A////AP///wD///8A////AP///wD///8AT0o/gIdzUP+QXwf/rnIC/7Z5
 A/+9gQP/y5AE/9ylBf/lsAb/6rYG/5tqA/+yoID/5+fn/9TV1P9nWT//zYcE/9yPAv/bjwL/2IwC
 /82FAv+9ewL/yIwe/9ubJ//loif/6qQk/9GRG/+2eg7/qYdK/7iwof/T0dD/qqqp/1xVR/8sMDUB
 ////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD/
 //8AeHFggP/////a2NX/sKmb/6mZfP+MZR//t30G/92mBf/msgb/5qgF/7R1A/+xoID/4eLi/25l
 Vf++fQP/0YgC/9qOAv/ekQL/3ZAC/+CaGf/quV//uJhb/6aUcv+ejnL/sqWO/8G4qP/c2tn/2tnY
 /9jX1f/W1NP/rayr/2BZS/////8A////AP///wD///8A////AP///wD///8A////AP///wD///8A
 ////AP///wD///8A////AP///wD///8AfXVlgP///////////////3laIf+/hQP/1J4F/9+rBv/e
 pwz/n41n/7uGJv/LxLT/cWxk/6BrB/+7egH/xYAB/9OJAv/fkQL/6qUo/+6/av++r5L/1dXU/9nY
 1//d3Nz/3t3c/93c2//b2tj/2dfW/9bV0//U0tH/rKuq/2NcT/////8A////AP///wD///8A////
 AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8AgXpqgP//////////
 /////7qoiP/BhwT/1qEF/9qnDv+qlmj/m5J8/5SJcP+0raD/TkIp/7F0Av+zdgH/uXkB/8iDAf/q
 qC7/37Rm/7ywnP/f39//3Nvb/93c3P/e3dz/3t3b/9za2f/Z2Nb/19XU/9XT0f/S0c//q6qp/2Zg
 Uv////8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////
 AP///wD///8Ahn9vgP////////////////Ty8P+lein/0Z0O/7Cec/+lnYn/opmE/5+WgP+ZkHn/
 cGZQ/0c3Gf+1dwH/tXYB/8OGFf/ku23/w7ei/+Xk4//j4uH/4eDf/9/e3f/d3Nv/3Nva/9rY1//X
 1tX/1dTS/9PR0P/Qz83/qqmo/2ljVv////8A////AP///wD///8A////AP///wD///8A////AP//
 /wD///8A////AP///wD///8A////AP///wD///8AioN0gP////////////////7+/v/a0sL/tXoQ
 /7Kig/+qo5D/p5+M/6Obh/+hmIT/lYx3/1VNOv9oSQ//t34T/8CaVv+UhGj/2tra/+Lh4P/i4eD/
 4eDf/9/e3P/d29r/2tnY/9jX1f/W1NP/09LQ/9HPzv/Ozcv/qain/21mWf////8A////AP///wD/
 //8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8Aj4h5gP//
 /////////v7+//39/f/8/Pz/0MWx/6KQav+tppb/pp6N/6egjv+8t6r/qKCN/2FZSP87NSj/alYx
 /3hnSP/CwsL/1NPT/9/e3f/g397/397d/93c2//b2dj/2dfW/9bV0//U0tH/0dDO/8/NzP/Ny8n/
 qKem/3BqXP////8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD/
 //8A////AP///wD///8Ak41+gP/////+/v7//f39//z8/P/7+/v/4d/a/3NpUv+jnY3/kYl3/66m
 lv/Hxb3/trCg/4V9bf9ZVUz/mZiV/7Gxsv+8vLz/z87O/9va2v/f3t3/3tzb/9va2f/Z2Nb/19XU
 /9TT0f/S0M//z87M/83Lyv/Lycf/p6al/3NtYP////8A////AP///wD///8A////AP///wD///8A
 ////AP///wD///8A////AP///wD///8A////AP///wD///8AmJGDgP/////+/v7//Pz8//v7+//w
 7+3/gXhi/2pgSP98dWP/l5KH/5aThv+vqJr/saqa/6yllP94dGv/u7u8/7Ozs/+6urr/y8rK/9jY
 1//d3Nv/3NvZ/9rY1//X1tT/1dPS/9PRz//Qzs3/zszK/8vJx//IxsT/pKOi/3ZwZP////8A////
 AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A
 nJaIgP7+/v/9/f3/+/v7//f29v+Xj33/bmRL/3JnT/9uZE7/jIh+/4yJgP+qpZj/ta6g/7Wunv+B
 e23/tLSy/7a2tv+5ubn/yMjH/9bU0//b2tn/2tnX/9jW1f/V1NP/09HQ/9HPzf/Ozcv/zMrI/8nH
 xf/Dwb//n52c/3l0Z/////8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////
 AP///wD///8A////AP///wD///8AoZuNgP7+/v/8/Pz/+vr6/7eypv9zaFD/d21V/3txWv98cl3/
 d3Fi/46NhP+dmpD/vLep/7y2qP+tp5n/gn95/7u7u/+3t7f/wsLC/9HQzv/Y1tX/2NfW/9bV0//U
 0tH/0dDO/8/Ny//Mysn/ysjG/8bEwv+5t7X/mZeW/313a/////8A////AP///wD///8A////AP//
 /wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8ApqCSgP7+/v/7+/v/0M3G
 /3huV/98clv/fXNe/4N6Zf+GfWr/Z2FS/3x5cv+joJf/vLes/8K9sf++uav/cm1i/7m4uP+3t7f/
 u7u7/8nIyP/U0tH/1tTT/9TT0f/S0M//z87M/83Lyf/Kycf/yMbE/768uv+tq6j/kpGP/4B6bv//
 //8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP//
 /wD///8AqqWYgP39/f/m5eH/iH9s/4F3Y/+FfGj/i4Ju/4+Gc/+NhnP/UExB/42Lhf+0sqr/vruy
 /8jDuf/GwbX/op2R/4WDff+4uLf/trW1/8HAwP/Ozcv/09HQ/9LRz//Qzs3/zczK/8vJx//IxsT/
 wsC+/7Oxrv+hnpv/jIqJ/4N+cv////8A////AP///wD///8A////AP///wD///8A////AP///wD/
 //8A////AP///wD///8A////AP///wD///8Ar6qdgPX19P+dlob/hn1q/4qCb/+RiXb/lo17/5OM
 ev9eWUz/o6Kd/8nJyP+7t7D/zcnC/8/Lwf/MyL7/x8O4/25pYP+2tbT/srKy/7m4uP/GxcT/z83M
 /8/OzP/OzMv/zMrI/8nHxf/Fw8H/uLa0/6ekov+UkY7/hoSC/4eBdv////8A////AP///wD///8A
 ////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8Aq6aZgLOtof+N
 hXL/kop4/5iQf/+clYT/mZKB/1BKPf+hoJr/2drZ/9/f3/+8urb/19TN/9XSyv/Tz8f/0MzD/42I
 gP+Rj4z/sbGw/7Kxsf+9vbv/ycfG/87Myv/Mysn/ysjG/8bEwv+9u7j/raqo/5uYlP+IhIH/f317
 /4qFev////8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A
 ////AP///wD///8AoJuNr5aOfP+YkYD/n5iH/6GbjP+gmov/YFtP/5iWkf/e3d7/5OTj/+bl5P/f
 3t3/xcK8/9za1P/Z19D/1tPL/8/Mw/9iXVX/s7Oy/66trf+1tLP/wcC//8nIxv/Jx8X/x8XD/8C+
 vP+zsK3/oZ6b/46LiP9ua2b/ZmRh/42Iff////8A////AP///wD///8A////AP///wD///8A////
 AP///wD///8A////AP///wD///8A////AP///wC6t65goJmJ/6Gaiv+lno//qaKU/6Sfkv9eWU//
 joyG/9/g3//m5uX/5+fm/+Xk5P/k4+L/v725/+Ph3f/g3tj/3drU/9nWz/+FgXn/lpSR/6ysq/+u
 raz/ubm3/8TCwf/GxMP/w8C+/7e1s/+Wk47/cm5o/1JNRv8+OjL/c3Fr/5SOgN////8A////AP//
 /wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////ALy5sjCrpZfvp6GT
 /6ummP+wq57/rKeb/1xXTf+JhoD/3Nva/+bl5P/o5+b/5+bl/+Xk4//j4uH/2tnY/8XDv//m5eH/
 4+Hd/+De2f/Ewbv/aWZf/66trf+pp6f/r6+u/6+sqv+gnZj/hIB4/2hiWf9cV07/UUxE/356df+y
 sK7/o5+U/5mThUD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP//
 /wD///8AuLauELSvpM+wqp7/ta+i/7WwpP+2sqf/eHRp/3d1bv/c29r/5ubl/+fn5v/m5eX/5eTj
 /+Pi4f/h4N//397d/7e1sf/t7Or/6unm/+bl4f/i4dz/d3Rs/6Wkov+mpaT/qKin/3FrY/96dm7/
 ko6H/7Kvq//W1NL//v7+/+7u7v+5t7P/oJqMr////wD///8A////AP///wD///8A////AP///wD/
 //8A////AP///wD///8A////AP///wD///8AtrOqj7m0qP+9uK3/wLuw/8C9s/+Hgnn/d3Rt/9jX
 1//m5eT/6Ofm/+fm5f/m5eT/5OPi/+Lg3//f3t3/3dzb/9TT0f+8urb/8PDu/+3s6v/q6eb/q6ij
 /3p4c/+npaT/oqGh/315dP/g4eH//Pz8////////////+Pj4/9DQ0P+nopbvpJ6QIP///wD///8A
 ////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8AvLiu/8XAtv/J
 xbz/y8e//5eSiv9va2P/09PR/+Li4f/n5+b/5+bm/+bl5P/k4+L/4uHg/+Df3v/e3Nv/29rZ/9nY
 1v+0sq//6+vq//Pz8v/w8O//6+rp/2ZiXf+rqqn/oJ+e/4iGg//U1NX/+/v7///////9/f3/5eXk
 /7KvqP+qpZdw////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A
 ////AP///wD///8AxcG5/9HNxf/W08v/wb64/3FtZP/W1dT/5eTj/+fm5f/o5+b/5+bl/+Xk4//j
 4uD/4N/e/97d3P/c29n/2tjX/9fW1P/V09L/trWy//X19f/19fX/8vLy/5KPiv+amZb/o6Gf/3l2
 cP/S09P/+/v7///////y8vL/wsG//66pnb////8A////AP///wD///8A////AP///wD///8A////
 AP///wD///8A////AP///wD///8A////AP///wD///8AxcO7/97b1f/Pzcj/fHhw/+Tj4f/t7ez/
 6Ojo/+jn5//m5eT/5eTj/+Pi4f/h4N//397c/9zb2v/a2dj/2NbV/9bU0//T0tD/rKqm/+vq6v/1
 9fX/9fX1/8rJxv+Bfnv/paSh/4OAev/S09P//Pz8//r6+v/Y2Nf/r6ui/7CrnjD///8A////AP//
 /wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8ApqOb
 j9vZ1f+XlY//p6OYv/f39//v7+7/6uno/+jn5v/m5eT/5OPi/+Hg3//f3t3/3dzb/9vZ2P/Y19b/
 1tXT/9TS0f/R0M7/z83L/66sqP/19fX/9fX1/9bV0/+Fgn3/paOg/4N/ev/S0tL//fz8/+rq6v+2
 tbD/sKuegP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP//
 /wD///8A////AP///wD///8A////AJeVjXCVkopgsKuegPf39v/u7e3/6Ofm/+bl5P/k4+L/4uHg
 /+Df3f/d3Nv/29rZ/9nY1v/X1dT/1NPR/9LQz//Pzsz/zcvJ/6mno//e3t3/9fX1/7Syrv+DgHv/
 n5uZ/4SAfP/Pz8//9PT0/8rJyP+vq5/P////AP///wD///8A////AP///wD///8A////AP///wD/
 //8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8AsKuegPb2
 9f/t7Ov/5+bl/+Tk4//i4eD/4N/e/97d3P/c2tn/2djX/9fW1P/V09L/0tHP/9DOzf/OzMr/y8nH
 /8jHxf+jop7/4eDf/3l2b/+dmpf/lJGN/3p2cv/IyMj/39/e/7Kvpu+wq54w////AP///wD///8A
 ////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD/
 //8A////AP///wD///8AsKuegPX19f/r6+r/5eTj/+Pi4f/h4N//3t3c/9zb2v/a2df/2NbV/9XU
 0v/T0dD/0c/N/87My//Mysj/ycfF/8XDwf+joZ7/amdh/5GOiv+alpP/jImF/397d/+5uLj/vry2
 /7CrnnD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A
 ////AP///wD///8A////AP///wD///8A////AP///wD///8AsKuegPX19P/4+Pj/+Pj3//f39//3
 9/b/9vb2//b19f/19fX/9fT0//T08//z8/P/8/Ly//Ly8f/y8fH/8fDw/+/u7v/t7Oz/6uno/9/e
 3f/Qz83/t7Wy/42MiP+4t7T/sKuen////wD///8A////AP///wD///8A////AP///wD///8A////
 AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A
 sKueQMC8td+9urP/vLmy/7q4sf+5trD/uLWu/7e0rf+2s6z/tLGr/7Owqf+yr6j/sK2n/6+spf+u
 q6T/rKmi/52alP+bmZL/mZeQ/5aUjf+Sj4n/jImC/5ORjL+0r6SP////AP///wD///8A////AP//
 /wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////
 AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A
 ////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD/
 //8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP//
 /wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////
 AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A
 ////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD/
 //8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP//
 /wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////
 AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///8A
 ////AP///wD///8A////AP///wD///8A////AP///wD///8A////AP///wD///////8AAP//////
 /wAA////////AAD/4D////8AAP/AH//D/wAA/4AAfgD/AAD/AAAAAD8AAP8AAAAAPwAA/wAAAAA/
 AAD/AAAAAD8AAP8AAAAAPwAA/wAAAAA/AAD/AAAAAD8AAP8AAAAAPwAA/4AAAAA/AAD/wAAAAD8A
 AP/AAAAAPwAA/8AAAAA/AAD/wAAAAD8AAP/AAAAAPwAA/8AAAAA/AAD/wAAAAD8AAP/AAAAAPwAA
 /8AAAAA/AAD/wAAAAD8AAP/AAAAAPwAA/8AAAAA/AAD/wAAAAD8AAP/AAAAAPwAA/8AAAAA/AAD/
 wAAAAD8AAP/AAAAAPwAA/8AAAAA/AAD/gAAAAH8AAP8AAAAAfwAA/gAAAAD/AAD+AAAAAf8AAP4A
 AAAB/wAA/gAAAAP/AAD+AAAAA/8AAP/AAAAH/wAA/8AAAA//AAD/wAAAH/8AAP/AAAAf/wAA/+AA
 AD//AAD///////8AAP///////wAA////////AAA=')
 	$formCreateNMEMasterList.Name = "formCreateNMEMasterList"
 	$formCreateNMEMasterList.Text = "Create NME Master List"
 	$formCreateNMEMasterList.add_Load($formCreateNMEMasterList_Load)
 	#
 	#
 	$checkboxUseWildcardSearchkey.Location = '13, 176'
 	$checkboxUseWildcardSearchkey.Name = "checkboxUseWildcardSearchkey"
 	$checkboxUseWildcardSearchkey.Size = '264, 24'
 	$checkboxUseWildcardSearchkey.TabIndex = 15
 	$checkboxUseWildcardSearchkey.Text = "Use wildcard search (*keyword*)"
 	$checkboxUseWildcardSearchkey.UseVisualStyleBackColor = $True
 	#
 	#
 	$labelNumberOfFilesToSplit.Location = '12, 322'
 	$labelNumberOfFilesToSplit.Name = "labelNumberOfFilesToSplit"
 	$labelNumberOfFilesToSplit.Size = '263, 23'
 	$labelNumberOfFilesToSplit.TabIndex = 14
 	$labelNumberOfFilesToSplit.Text = "Number of files to split into:"
 	#
 	#
 	$ColumnsForKey.DropDownStyle = 'DropDownList'
 	$ColumnsForKey.FormattingEnabled = $True
 	$ColumnsForKey.Location = '12, 92'
 	$ColumnsForKey.Name = "ColumnsForKey"
 	$ColumnsForKey.Size = '228, 21'
 	$ColumnsForKey.TabIndex = 13
 	#
 	#
 	$labelColumnToUseAsKeyforV.Location = '13, 66'
 	$labelColumnToUseAsKeyforV.Name = "labelColumnToUseAsKeyforV"
 	$labelColumnToUseAsKeyforV.Size = '264, 23'
 	$labelColumnToUseAsKeyforV.TabIndex = 12
 	$labelColumnToUseAsKeyforV.Text = "Column to use as key: (for values in the file below)"
 	#
 	#
 	$FileBuildingProgress.Location = '0, 428'
 	$FileBuildingProgress.Name = "FileBuildingProgress"
 	$FileBuildingProgress.Size = '289, 23'
 	$FileBuildingProgress.TabIndex = 11
 	#
 	#
 	$statusbar.Location = '0, 451'
 	$statusbar.Name = "statusbar"
 	$statusbar.Size = '289, 22'
 	$statusbar.TabIndex = 10
 	$statusbar.Text = "Ready"
 	#
 	#
 	$buttonBuildFiles.Location = '13, 379'
 	$buttonBuildFiles.Name = "buttonBuildFiles"
 	$buttonBuildFiles.Size = '264, 43'
 	$buttonBuildFiles.TabIndex = 9
 	$buttonBuildFiles.Text = "Build file(s)"
 	$buttonBuildFiles.UseVisualStyleBackColor = $True
 	$buttonBuildFiles.add_Click($buttonBuildFiles_Click)
 	#
 	#
 	$buttonBrowseFolder.Location = '246, 249'
 	$buttonBrowseFolder.Name = "buttonBrowseFolder"
 	$buttonBrowseFolder.Size = '30, 23'
 	$buttonBrowseFolder.TabIndex = 4
 	$buttonBrowseFolder.Text = "..."
 	$buttonBrowseFolder.UseVisualStyleBackColor = $True
 	$buttonBrowseFolder.add_Click($buttonBrowseFolder_Click2)
 	#
 	#
 	$textboxFolder.AutoCompleteMode = 'SuggestAppend'
 	$textboxFolder.AutoCompleteSource = 'FileSystemDirectories'
 	$textboxFolder.Location = '12, 251'
 	$textboxFolder.Name = "textboxFolder"
 	$textboxFolder.Size = '228, 20'
 	$textboxFolder.TabIndex = 3
 	$textboxFolder.add_TextChanged($textboxFolder_TextChanged)
 	#
 	#
 	$NumberOfFiles.DropDownStyle = 'DropDownList'
 	$NumberOfFiles.Enabled = $False
 	$NumberOfFiles.FormattingEnabled = $True
 	$NumberOfFiles.Location = '12, 348'
 	$NumberOfFiles.Name = "NumberOfFiles"
 	$NumberOfFiles.Size = '129, 21'
 	$NumberOfFiles.TabIndex = 7
 	#
 	#
 	$checkboxSplitTheFilesForMe.Enabled = $False
 	$checkboxSplitTheFilesForMe.Location = '13, 295'
 	$checkboxSplitTheFilesForMe.Name = "checkboxSplitTheFilesForMe"
 	$checkboxSplitTheFilesForMe.Size = '264, 24'
 	$checkboxSplitTheFilesForMe.TabIndex = 6
 	$checkboxSplitTheFilesForMe.Text = "Split the files for me!"
 	$checkboxSplitTheFilesForMe.UseVisualStyleBackColor = $True
 	$checkboxSplitTheFilesForMe.add_CheckedChanged($checkboxSplitTheFilesForMe_CheckedChanged)
 	#
 	#
 	$labelSaveFilesTo.Location = '12, 225'
 	$labelSaveFilesTo.Name = "labelSaveFilesTo"
 	$labelSaveFilesTo.Size = '100, 23'
 	$labelSaveFilesTo.TabIndex = 5
 	$labelSaveFilesTo.Text = "Save file(s) to:"
 	#
 	#
 	$checkboxMigrateAllUsersusedT.Location = '13, 198'
 	$checkboxMigrateAllUsersusedT.Name = "checkboxMigrateAllUsersusedT"
 	$checkboxMigrateAllUsersusedT.Size = '264, 24'
 	$checkboxMigrateAllUsersusedT.TabIndex = 3
 	$checkboxMigrateAllUsersusedT.Text = "Migrate all users (used to only split the master file)"
 	$checkboxMigrateAllUsersusedT.UseVisualStyleBackColor = $True
 	$checkboxMigrateAllUsersusedT.add_CheckedChanged($checkboxMigrateAllUsersusedT_CheckedChanged)
 	#
 	#
 	$buttonUserMigrateBrowse.Location = '247, 147'
 	$buttonUserMigrateBrowse.Name = "buttonUserMigrateBrowse"
 	$buttonUserMigrateBrowse.Size = '30, 23'
 	$buttonUserMigrateBrowse.TabIndex = 1
 	$buttonUserMigrateBrowse.Text = "..."
 	$buttonUserMigrateBrowse.UseVisualStyleBackColor = $True
 	$buttonUserMigrateBrowse.add_Click($buttonUserMigrateBrowse_Click)
 	#
 	#
 	$textboxUserToMigrate.AutoCompleteMode = 'SuggestAppend'
 	$textboxUserToMigrate.AutoCompleteSource = 'FileSystem'
 	$textboxUserToMigrate.Location = '13, 149'
 	$textboxUserToMigrate.Name = "textboxUserToMigrate"
 	$textboxUserToMigrate.Size = '228, 20'
 	$textboxUserToMigrate.TabIndex = 0
 	$textboxUserToMigrate.add_TextChanged($textboxUserToMigrate_TextChanged)
 	#
 	#
 	$labelUsersToMigratevalues.Location = '13, 123'
 	$labelUsersToMigratevalues.Name = "labelUsersToMigratevalues"
 	$labelUsersToMigratevalues.Size = '264, 23'
 	$labelUsersToMigratevalues.TabIndex = 2
 	$labelUsersToMigratevalues.Text = "Users to migrate: (values matching the above key)"
 	#
 	#
 	$buttonMasterListBrowse.Location = '247, 37'
 	$buttonMasterListBrowse.Name = "buttonMasterListBrowse"
 	$buttonMasterListBrowse.Size = '30, 23'
 	$buttonMasterListBrowse.TabIndex = 1
 	$buttonMasterListBrowse.Text = "..."
 	$buttonMasterListBrowse.UseVisualStyleBackColor = $True
 	$buttonMasterListBrowse.add_Click($buttonMasterListBrowse_Click)
 	#
 	#
 	$textboxMasterTSV.AutoCompleteMode = 'SuggestAppend'
 	$textboxMasterTSV.AutoCompleteSource = 'FileSystem'
 	$textboxMasterTSV.Location = '13, 39'
 	$textboxMasterTSV.Name = "textboxMasterTSV"
 	$textboxMasterTSV.Size = '228, 20'
 	$textboxMasterTSV.TabIndex = 0
 	$textboxMasterTSV.add_TextChanged($textboxMasterTSV_TextChanged)
 	#
 	#
 	$labelSourceMasterTsv.Location = '13, 13'
 	$labelSourceMasterTsv.Name = "labelSourceMasterTsv"
 	$labelSourceMasterTsv.Size = '262, 23'
 	$labelSourceMasterTsv.TabIndex = 0
 	$labelSourceMasterTsv.Text = "Source master tsv:"
 	#
 	#
 	$openfiledialog1.DefaultExt = "txt"
 	$openfiledialog1.Filter = "Text File (.tsv)|*.tsv|All Files|*.*"
 	$openfiledialog1.ShowHelp = $True
 	#
 	#
 	$openfiledialog2.DefaultExt = "txt"
 	$openfiledialog2.Filter = "Text File (.txt)|*.txt|All Files|*.*"
 	$openfiledialog2.ShowHelp = $True
 	#
 	#
 	#
 	#
 
 	#----------------------------------------------
 
 	$InitialFormWindowState = $formCreateNMEMasterList.WindowState
 	$formCreateNMEMasterList.add_Load($Form_StateCorrection_Load)
 	$formCreateNMEMasterList.add_FormClosed($Form_Cleanup_FormClosed)
 	return $formCreateNMEMasterList.ShowDialog()
 
 
 if((OnApplicationLoad) -eq $true)
 {
 	Call-CreateMasterLists_pff | Out-Null
 	OnApplicationExit
 }
`

