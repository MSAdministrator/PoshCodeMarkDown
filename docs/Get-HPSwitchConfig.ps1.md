---
Author: jgrote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5888
Published Date: 2015-06-09t19
Archived Date: 2015-06-15t00
---

# get-hpswitchconfig - 

## Description

used for fetching an hp procurve switch config via sftp for backup and revision control. made this because pcm is no longer free. very basic error checking, use at own risk.

## Comments



## Usage



## TODO



## module

`get-hpswitchconfig`

## Code

`#
 #
 function get-HPSwitchConfig {
 <#
 
 .SYNOPSIS
 Retrieve configuration from an HP Procurve Switch via SFTP, store, and timestamp it in a repository.
 
 .NOTES
 WARNING: Copying config via SFTP has been observed to interrupt the data forwarding on a 2810 switch for a few seconds. ONLY PERFORM OFF HOURS
 This requires the POSH-SSH powershell module (https://github.com/darkoperator/Posh-SSH) 
 You also need to enable SFTP on the switch via "ip ssh filetransfer" command before this will work.
 Also it will not work if you already have 4 concurrent management sessions to your switch as it counts as a management session.
 Make sure you don't have any SSH or SFTP sessions open to the switch you are about to pull from, some HP switches don't allow more than 1 SSH session from a host at a time.
 
 .PARAMETER switchHost
 
 DNS Name or IP address of the switch to connect to. If multiple strings are passed by pipeline, this will retrieve the config for each switch
 
 .PARAMETER startupconfig
 
 Retrieves the switch's startup config instead of the running config
 
 .EXAMPLE 
 
 Get-HPSwitchConfig -switchhost "switch1"
 
 Retrieve running config from switch
 
 .EXAMPLE 
 "switch1","switch2","switch3","switch4" | get-hpswitchconfig -startupconfig
 Gets the Startup config from 4 different switches
 #>
 
 param (
         [parameter(Mandatory=$true,ValuefromPipeline=$true)][string]$switchHost,
         [PSCredential]$switchCredential = (get-credential -message "Enter Procurve SSH Admin Credentials"),
         [int]$timeout = 10,
 )
 
 begin {
     $erroractionpreference = "stop"
     if (-Not (test-path $RepositoryPath)) {throw "Cannot find Output Path $RepositoryPath. Please ensure it exists."}
 
     $cfgFile="running-config"
     if ($startupconfig) {$cfgFile = "startup-config"}
     $cfgFileSourcePath="/cfg/$cfgfile"
     $cfgOutputPath = new-item -ItemType Directory -path "$RepositoryPath\$switchhost" -force
 
 
     if (get-sftpsession | where {$_.host -match $switch}) {$removedsession=remove-sftpsession (get-sftpsession | where {$_.host -match $switch})}
 
     try {
         $sftpSession = new-sftpsession $switch -credential $switchcredential -connectiontimeout $timeout -operationtimeout $timeout 
         if (-Not $sftpsession.connected) {write-error "Could not connect to $switch. Please check connectivity and credentials.";return}
 
         get-sftpfile $sftpsession -RemoteFile $cfgFileSourcePath -LocalPath $cfgOutputPath -overwrite 
         if (-Not (test-path $cfgoutputpath\$cfgfile)) {throw "File not downloaded"}
     catch [Exception] {write-error "Error while downloading $switchHost configuration, please ensure you have enabled IP SSH Fileserver and do not have multiple sessions or current SSH/SFTP sessions from this host in other programs";return}
     
     remove-sftpsession $sftpSession
     $cfgfilepath = copy-item "$cfgOutputPath\$cfgfile" -destination ("$cfgOutputPath\$cfgfile-" + ((get-date -format s) -replace ':','') + ".hpconfig") -passthru
 
     $objStatusProps = @{}
         $objStatusProps.Switch = $switch
         $objStatusProps.ConfigFilePath = $cfgfilepath.fullname
     $objStatus = new-object -TypeName PSObject -property $objStatusProps
     return $objStatus
 
 
 
 
 
`

