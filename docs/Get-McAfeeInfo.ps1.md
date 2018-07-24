---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 8.0
Encoding: ascii
License: cc0
PoshCode ID: 6245
Published Date: 2016-03-02t14
Archived Date: 2016-03-26t10
---

# get-mcafeeinfo - 

## Description

this script produces report for mcafee antivirus on a set of remote computers. writes output object containing server, product version, engine version, and virus definition version.

## Comments



## Usage

get-content <list of servers.txt> | .\getmcafeeinfo.ps1

## TODO



## function

``

## Code

`#
 #
 #------------------------------------------------------------------
 #------------------------------------------------------------------
 
 Begin {
 	function SelectAlive ( $cn ) {
 		$reply = $null
 		$reply = $ping.Send($cn)
 		if ($reply.status -eq "Success") {
 			Write-Output $cn
 		} else {
 			Write-Warning "$cn : not pingable"
 		}
 	}
 
 	function SelectRemoteRegistryAccess ( $cn ) {
 		$root = "LocalMachine"
 		if ( [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey( $root, $cn ) ) {
 			Write-Output $cn
 		} else {
 			Write-Warning "$cn : remote registry access not accessible"
 		}
 	}
 	
 	function GetHKLMRegValue ( $cn, $key, $value ) {
 		$root = "LocalMachine"
 		$rootkey = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey( $root, $cn )
 		$key = $rootkey.OpenSubKey( $key )
 		$key.GetValue( $value )
 	}
 	
 	$sKeyPath85 = "SOFTWARE\McAfee\DesktopProtection"
 	$sKeyPath85e = "SOFTWARE\McAfee\AVEngine"
 	$sKeyPath80 = "SOFTWARE\Network Associates\TVD\VirusScan Enterprise\CurrentVersion"
 	$sProductVer = "szProductVer"
 	$sEngineVer80 = "szEngineVer"
 	$sEngineVer85 = "EngineVersionMajor"
 	$sVirDefVer80 = "szVirDefVer"
 	$sVirDefVer85 = "AVDatVersion"
 
 }
 Process {
 	if ( -not ( SelectAlive $_ ) ) { continue }
 	
 	if ( -not ( SelectRemoteRegistryAccess $_ ) ) { continue }
 	
 	$Output = "" | Select-Object Server, ProductVersion, EngineVersion, DatVersion
 	
 	if ( GetHKLMRegValue -cn $_ -key $sKeyPath85 -value $sProductVer ) {
 		$output.Server = $_
 		$output.ProductVersion = GetHKLMRegValue -cn $_ -key $sKeyPath85 -value $sProductVer
 		$output.EngineVersion = GetHKLMRegValue -cn $_ -key $sKeyPath85 -value $sEngineVerValue
 		$output.DatVersion = GetHKLMRegValue -cn $_ -key $sKeyPath85 -value $sVirDefVer85
 	} else {
 	}
 	$output
 }
`

