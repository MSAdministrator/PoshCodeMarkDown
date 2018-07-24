---
Author: dan smith
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 2897
Published Date: 2012-08-05t22
Archived Date: 2017-05-11t14
---

# ie proxy toggle (silent) - 

## Description

immediately toggle the current user’s internet explorer proxy settings on/off. uses a ‘hidden’ internet explorer process to trigger the application of the new proxy setting once its been changed in the registry. accepts a single command line parameter “disable”.  if no parameter is given, the proxy is “enabled”.

## Comments



## Usage



## TODO



## function

`start-hiddenieprocess`

## Code

`#
 #
 	[bool] $DISABLE_PROXY = $false;
     	foreach ($param in $MyInvocation.UnboundArguments) {
 		if ($param -like 'disable') { $DISABLE_PROXY = $true; }
 	}
 	
 	[string] $proxyServer   = '###.###.###.###:####';
 	[string] $proxyOverride = '*.local;192.168.*;<local>';
 	[int]    $migrateProxy  = 1;
 	[int]    $proxyHttp11   = 1;
 	
 	REG ADD """HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"" /v ""ProxyServer"" /t REG_SZ /d ""$proxyServer"" /f" | Out-Null;
 	REG ADD """HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"" /v ""ProxyOverride"" /t REG_SZ /d ""$proxyOverride"" /f" | Out-Null;
 	REG ADD """HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"" /v ""MigrateProxy"" /t REG_DWORD /d $migrateProxy /f" | Out-Null;
 	REG ADD """HKCU\Software\Microsoft\Windows\CurrentVersion\Internet Settings"" /v ""ProxyHttp1.1"" /t REG_DWORD /d $proxyHttp11 /f" | Out-Null;
 
 Define a function to start a 'hidden' Internet Explorer process. 
 This is needed since there is no easy way to programmatically 'refresh'
 the user's proxy settings after they have been changed in the registry.
 #>
 function Start-HiddenIEProcess () {
 	[object] $ieProcess = $null;
 	
 		if (Test-Path "$env:PROGRAMFILES(X86)\Internet Explorer\iexplore.exe") {	
 			$ieProcess = Start-Process -Passthru -FilePath "$env:PROGRAMFILES(X86)\Internet Explorer\iexplore.exe" -WindowStyle Hidden;
 		} else {		
 			$ieProcess = Start-Process -Passthru -FilePath "$env:PROGRAMFILES\Internet Explorer\iexplore.exe" -WindowStyle Hidden;
 		}
 		[int] $count = 10;
 		do {
 			if (($ieProcess.WaitForInputIdle())) {
 				$count   = 0;
 			} else { 
 			}
 		} while ($waiting -or ($count -gt 0));
 		
 		if ((Get-Process -Name iexplore -ErrorAction SilentlyContinue)) {
 			return $true;
 		} else {
 			return $false;
 		}
 
 Define a function to toggle the user's proxy settings by altering the
 required registry entry and then forcing a 'refresh' of the proxy status
 by utilizing a hidden/temporary Internet Explorer process. 
 #>
 function Set-ProxyState ([bool] $disable) {
 		Stop-Process -Name iexplore -Force -ErrorAction SilentlyContinue;
 		
 		if ($disable) {
 			Write-Host 'DISABLING user''s proxy settings...';
 		} else {
 			Write-Host 'ENABLING user''s proxy settings...';
 		}
 		
 		Start-HiddenIEProcess;
 		
 		Stop-Process -Name iexplore -Force -ErrorAction SilentlyContinue;
 		
 	Write-Host '[DONE]';
 		
 	return $null;
 
 	$null = Set-ProxyState $DISABLE_PROXY;
`

