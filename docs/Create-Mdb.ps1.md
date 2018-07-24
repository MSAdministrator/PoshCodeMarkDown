---
Author: justin dearing
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: mitl
PoshCode ID: 2928
Published Date: 2012-08-25t04
Archived Date: 2012-02-02t06
---

# create-mdb.ps1 - 

## Description

a script that demonstrates how to create a microsoft access database in powershell without microsoft access installed. it works in 32 and 64 bit instances of powershell.

## Comments



## Usage



## TODO



## script

`create-mdb`

## Code

`#
 #
 #
 
 
 
 
 Add-Type -TypeDefinition @"
 namespace PInvoke {
 	public enum ODBC_Constants {
 		ODBC_ADD_DSN = 1,
 		ODBC_CONFIG_DSN,
 		ODBC_REMOVE_DSN,
 		ODBC_ADD_SYS_DSN,
 		ODBC_CONFIG_SYS_DSN,
 		ODBC_REMOVE_SYS_DSN,
 		ODBC_REMOVE_DEFAULT_DSN,
 	};
 
 	public enum SQL_RETURN_CODE {
 		SQL_ERROR = -1,
 		SQL_INVALID_HANDLE = -2,
 		SQL_SUCCESS = 0,
 		SQL_SUCCESS_WITH_INFO = 1,
 		SQL_STILL_EXECUTING = 2,
 		SQL_NEED_DATA = 99,
 		SQL_NO_DATA = 100
 	}
 }
 "@;
 
 $signature = @'
 [DllImport("ODBCCP32.DLL",CharSet=CharSet.Unicode, SetLastError=true)]
 public static extern int SQLConfigDataSource 
 	(int hwndParent, int fRequest, string lpszDriver, string lpszAttributes);
 
 [DllImport("odbccp32", CharSet=CharSet.Auto)]
 public static extern int SQLInstallerError(int iError, ref int pfErrorCode, StringBuilder lpszErrorMsg, int cbErrorMsgMax, ref int pcbErrorMsg);
 '@;
 
 Add-Type -MemberDefinition $signature -Name Win32Utils -Namespace PInvoke -Using PInvoke,System.Text;
 
 Function Create-MDB ([string] $fileName, [switch] $DeleteIfExists = $false) {
 	$fileName = [System.IO.Path]::GetFullPath($fileName);
 	if ($DeleteIfExists -and (Test-Path $fileName)) {
 		Remove-Item $fileName;
 	}
 	[string] $attrs = [string]::Format("CREATE_DB=`"{0}`" General`0", $fileName);
 	[string] $driver = 'Microsoft Access Driver (*.mdb)';
 	if ([IntPtr]::Size -eq 8) { 
 		$driver = 'Microsoft Access Driver (*.mdb, *.accdb)';
 	}
 	[int] $retCode = [PInvoke.Win32Utils]::SQLConfigDataSource(
 		0, [PInvoke.ODBC_Constants]::ODBC_ADD_DSN, 
 		$driver, $attrs);
 	if ($retCode -eq 0) {
 		[int] $errorCode = 0 ;
 		[int]  $resizeErrorMesg = 0 ;
 		$sbError = New-Object System.Text.StringBuilder(512);
 	[PInvoke.Win32Utils]::SQLInstallerError(1, [ref] $errorCode, $sbError, $sbError.MaxCapacity, [ref] $resizeErrorMesg);
 	if ($sbError.ToString() -eq 'Component not found in the registry') {
 		$sbError = New-Object System.Text.StringBuilder(512);
 		if ([intptr]::Size -eq 8) { $sbError.Append("Their appears to be no 64 bit MS Access Driver installed. Please install a 64 bit access driver or run this script from a 32 bit powershell instance"); }
 		if ([intptr]::Size -eq 4) { $sbError.Append("Their appears to be no 32 bit MS Access Driver installed. Please install a 32 bit access driver or run this script from a 64 bit powershell instance"); }
 		}
 		throw New-Object ApplicationException([string]::Format("Cannot create file: {0}. Error: {1}", $fileName, $sbError));
 	}
 }
 
 
 Create-Mdb ".\foo.mdb" -DeleteIfExists
`

