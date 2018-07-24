---
Author: anti121
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1985
Published Date: 2011-07-19t17
Archived Date: 2017-03-31t03
---

# promptfor-file - 

## Description

function to prompt user for file path input. can create a open file type dialog or a save file type dialog by either specifying either “open” or “save” for the -type parameter. either a file path string is returned or null if the user canceled the dialog.

## Comments



## Usage



## TODO



## function

`promptfor-file`

## Code

`#
 #
 function PromptFor-File 
 {
 	param
 	(	
 		[String] $Type = "Open",
 		[String] $Title = "Select File",
 		[String] $Filename = $null,
 		[String[]] $FileTypes,
 		[switch] $RestoreDirectory,
 		[IO.DirectoryInfo] $InitialDirectory = $null
 	)
 	
 	[void][System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms')
 	
 	if ($FileTypes)
 	{
 		$FileTypes | % {
 			$filter += $_.ToUpper() + " Files|*.$_|"
 		}
 		$filter = $filter.TrimEnd("|")
 	}
 	else
 	{
 		$filter = "All Files|*.*"
 	}
 	
 	switch ($Type)
 	{
 		"Open" 
 		{
 			$dialog = New-Object System.Windows.Forms.OpenFileDialog
 			$dialog.Multiselect = $false
 		}
 		"Save"
 		{
 			$dialog = New-Object System.Windows.Forms.SaveFileDialog
 		}
 	}
 	
 	$dialog.FileName = $Filename
 	$dialog.Title = $Title
 	$dialog.Filter = $filter
 	$dialog.RestoreDirectory = $RestoreDirectory
 	$dialog.InitialDirectory = $InitialDirectory.Fullname
 	$dialog.ShowHelp = $true
 	
 	if ($dialog.ShowDialog() -eq [System.Windows.Forms.DialogResult]::OK)
 	{
 		return $dialog.FileName
 	}
 	else
 	{
 		return $null
 	}
 }
`

