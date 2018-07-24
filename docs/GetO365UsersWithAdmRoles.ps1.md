---
Author: julien labalme
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5965
Published Date: 2016-08-04t10
Archived Date: 2016-05-17t16
---

# geto365userswithadmroles - 

## Description

this script will help you report which users have whic roles in the o365 platform

## Comments



## Usage



## TODO



## function

`convert-csvtoexcel`

## Code

`#
 #
 #############################################################################
 #
 #
 #############################################################################
 
 
 $Date=get-date
 $Date = $Date.Year.tostring() + "-" + $Date.Month.tostring() + "-" + $Date.Day.tostring() + "_" + $Date.Hour.tostring() + "-" + $Date.Minute.tostring()
 $LogFilePath = "$env:USERPROFILE\Desktop\O365UsersWithAdminRights - $Date.csv"
 $ExcelLogFilePath = "$env:USERPROFILE\Desktop\O365UsersWithAdminRights - $Date.xlsx"
 
 $Office365Account = ""
 $Office365Password = ""
 
 function writelog ([string]$text, [int] $color=0)
 {
 	Write-Output  $text | Out-File -Append -FilePath $LogFilePath -Encoding "UTF8"
 	if ($color -eq 0) {write-host $text}
 	if ($color -eq 1) {write-host $text -foregroundcolor green}
 	if ($color -eq 2) {write-host $text -foregroundcolor yellow}
 	if ($color -eq 3) {write-host $text -foregroundcolor red}
 	if ($color -eq 4) {write-host $text -foregroundcolor cyan}
 }	
 
 Function ConnectOffice365
 {
 	$Connected = $false
 	#{
 		try
 		{
 			$secstr = New-Object -TypeName System.Security.SecureString -ea stop
 			$Office365Password.ToCharArray() | ForEach-Object {$secstr.AppendChar($_)} -ea stop
 			$Onlinecred = new-object -typename System.Management.Automation.PSCredential -argumentlist $Office365Account, $secstr -ea Stop
 			Import-Module MSOnline
 			Connect-MsolService -Credential $Onlinecred
 			$Connected = $true
 		}
 		catch{writelog "ERROR,N/A,ConnectOffice365,$($ERROR[0])" 3}
 	#}
 	return $Connected
 }
 
 Function Convert-CSVToExcel
 {
 	[CmdletBinding()]
 	Param
 	(
 		[Parameter(ValueFromPipeline=$True, Position=0, Mandatory=$True, HelpMessage="Name of CSV/s to import")]
 		[ValidateNotNullOrEmpty()]
 		[string]$Inputfile,
 
 		[Parameter(ValueFromPipeline=$False, Position=1, Mandatory=$True, HelpMessage="Name of excel file output")]
 		[ValidateNotNullOrEmpty()]
 		[string]$Outputfile
 	)
 
     Begin
 	{
         If (!(Test-Path -Path $Inputfile))
         {
 	        write-host "CSV file not found:  {0}"
 	        Exit
         } 
 		
         $Excel = new-object -com excel.application
         $Excel.DisplayAlerts = $False 
         $Excel.ScreenUpdating = $False 
         $Excel.Visible = $False 
         $Excel.UserControl = $False 
         $Excel.Interactive = $False
         $workbook = $Excel.workbooks.Add()
     }
 
     Process
 	{
         $worksheet = $workbook.worksheets.Item(1)
         #$worksheet.name = "$((GCI $input).basename)"
         $tempcsv = $Excel.Workbooks.Open($Inputfile) 
         $tempsheet = $tempcsv.Worksheets.Item(1)
         $tempSheet.UsedRange.Copy() | Out-Null
         $worksheet.Paste()
         $tempcsv.close()
 
         $worksheet.Name = 'O365 Admin Rights';
         $range = $worksheet.Range("A1:C1");
         $range.Interior.ColorIndex = 43;
         $range.Font.ColorIndex = 1;
         $range.Font.Bold = $True;
         $range = $worksheet.UsedRange;
 		$range.EntireColumn.AutoFit() | out-null
 		$range.Cells.EntireColumn.AutoFilter();
     }        
 	
     End
 	{
         $workbook.saveas("$Outputfile")
         $Null = Start-sleep -Seconds 2
 		$OutputFile = $OutputFile.Substring(($OutputFile.LastIndexOf("\")+1),($OutputFile.Length-$OutputFile.LastIndexOf("\")-1))
 		write-host "CSV file $OutputFile successfully converted to XLSX"
 		
         $Excel.Quit()
 		
 		while( [System.Runtime.Interopservices.Marshal]::ReleaseComObject($Excel)){}
 		Remove-Variable Excel
     }
 }
 
 
 if (ConnectOffice365)
 {
 	writelog "ROLE;DISPLAYNAME;EMAILADDRESS"
 
 	$roles = Get-MsolRole | select ObjectId,Name
 	
 	Foreach ($role in $roles)
 	{
 		$roleMembers = Get-MSOLRoleMember -RoleObjectId $role.ObjectId | select EmailAddress,DisplayName
 		
 		Foreach ($roleMember in $roleMembers)
 		{
 			writelog "$($role.Name);$($roleMember.DisplayName);$($roleMember.EmailAddress)"
 		}
 	}
 }
 	
 $Null = Convert-CSVToExcel -Inputfile $LogFilePath -Outputfile $ExcelLogFilePath
 
 Remove-Item -Path $LogFilePath -Force -Confirm:$False
`

