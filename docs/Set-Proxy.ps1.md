---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3795
Published Date: 2012-11-28t18
Archived Date: 2012-12-03t17
---

# set-proxy - 

## Description

get and or set proxy thru remote registry

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 param(
 [Parameter(Position=0,ValueFromPipeline=$True,ValueFromPipelineByPropertyName=$true)]
 [alias("Name","ComputerName")] $Computer = @($env:computername),
 [switch] $ClearProxy
 )
 
 begin{
 $IEsettingsKey = ".DEFAULT\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
 [array]$ProxyResult = @()
 
 function SetProxySettings($Computer){
 	Set-RegDWord -ComputerName $Computer -Hive Users -Key $IEsettingsKey -Value ProxyEnable -Data 0 -Force
 	Set-RegString -ComputerName $Computer -Hive Users -Key $IEsettingsKey -Value ProxyServer -Force
 	Set-RegString -ComputerName $Computer -Hive Users -Key $IEsettingsKey -Value ProxyOverride -Force
 	Set-RegBinary -ComputerName $Computer -Hive Users -Key "$IEsettingsKey\Connections" -Value DefaultConnectionSettings -Force
 	Set-RegBinary -ComputerName $Computer -Hive Users -Key "$IEsettingsKey\Connections" -Value SavedLegacySettings -Force
 }
 
 function GetProxySettings($Computer){
 	$Output = @()
 	$Output += (Get-RegDWord -ComputerName $Computer -Hive Users -Key $IEsettingsKey -Value "ProxyEnable" | Select-Object ComputerName,Value,Data)
 	$Output += (Get-RegString -ComputerName $Computer -Hive Users -Key $IEsettingsKey -Value "ProxyServer" | Select-Object ComputerName,Value,Data)
 	$Output += (Get-RegString -ComputerName $Computer -Hive Users -Key $IEsettingsKey -Value "ProxyOverride" | Select-Object ComputerName,Value,Data)
 	$Output += (Get-RegBinary -ComputerName $Computer -Hive Users -Key "$IEsettingsKey\Connections" -Value "DefaultConnectionSettings" | Select-Object ComputerName,Value,Data)
 	$Output += (Get-RegBinary -ComputerName $Computer -Hive Users -Key "$IEsettingsKey\Connections" -Value "SavedLegacySettings" | Select-Object ComputerName,Value,Data)
 	return $Output
 }
 }
 
 process{
 if (Test-connection $Computer -quiet -count 1){
 	Try {
 		$ProxyResult += GetProxySettings $Computer
 		if ($ClearProxy){SetProxySettings $Computer; $ProxyResult += GetProxySettings $Computer}
 		}
 	Catch {
 		Write-Warning "$($error[0]) "
 		Break
 		}
 	}
 }
 
 end{
 $ProxyResult
 }
`

