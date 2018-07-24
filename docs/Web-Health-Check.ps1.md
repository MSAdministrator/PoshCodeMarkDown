---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1381
Published Date: 
Archived Date: 2009-10-26t15
---

# web health check - 

## Description

web-health-check.ps1 is a powershell script to monitor a website and automatically take remedial action if the site is not functioning properly. it works by retrieving a web page from a specified url and checking the page for the presence of a specific piece of text.

## Comments



## Usage



## TODO



## script

`send-smtpmail`

## Code

`#
 #
 #
 #
 param( 	$VISRV,
 		$url,
 		$CHECKVM,
 		$testpattern)
 #
 #
 if ( !$CHECKVM ) { $CHECKVM = "vm-name" }
 if ( !$url ) { $url = "http://www.thehypervisor.com/web-health-check/" }
 if ( !$testpattern ) { $testpattern = "©" }
 $toFile = "c:\test.html"
 #
 $SMTPSRV = "my-smtp.example.com"
 #
 $EmailFrom = "my-server@example.com"
 #
 $EmailTo = "me@example.com"
 $SetUsername = "administrator"
 #
 $CredFile = "c:\mycred.crd"
 #######################################
 #######################################
 if ($VISRV -eq ""){
 	Write-Host
 	Write-Host "Please specify a VI Server name eg...."
 	Write-Host "      powershell.exe webserv-healthcheck.ps1 MYVISERVER"
 	Write-Host
 	Write-Host
 	exit
 }
 
 function Send-SMTPmail($to, $from, $subject, $smtpserver, $body) {
 	$mailer = new-object Net.Mail.SMTPclient($smtpserver)
 	$msg = new-object Net.Mail.MailMessage($from,$to,$subject,$body)
 	$msg.IsBodyHTML = $true
 	$mailer.send($msg)
 }
 
 function Find-Username ($username){
 	if ($username -ne $null)
 	{
 		$root = [ADSI]""
 		$filter = ("(&(objectCategory=user)(samAccountName=$Username))")
 		$ds = new-object  system.DirectoryServices.DirectorySearcher($root,$filter)
 		$ds.PageSize = 1000
 		$ds.FindOne()
 	}
 }
 
 Function Set-Cred ($File) {
 	$Credential = Get-Credential
 	$credential.Password | ConvertFrom-SecureString | Set-Content $File
 }
 
 Function Get-Cred ($User,$File) {
 	$password = Get-Content $File | ConvertTo-SecureString
 	$credential = New-Object System.Management.Automation.PsCredential($user,$password)
 	$credential
 }
 
 If ($SetUsername -ne ""){
 	if ((Test-Path -Path $CredFile) -eq $false) {
 		Set-Cred $CredFile
 	}
 	$creds = Get-Cred $SetUsername $CredFile
 }
 Add-PSSnapin VMware.VimAutomation.Core
 $VIServer = Connect-VIServer $VISRV
 If ($VIServer.IsConnected -ne $true){
 	$USER = $env:username
 	$APPPATH = "C:\Documents and Settings\" + $USER + "\Application Data"
 
 	if ($env:appdata -eq $null -or $env:appdata -eq 0)
 	{
 		$env:appdata = $APPPATH
 	}
 	$VIServer = Connect-VIServer $VISRV
 	If ($VIServer.IsConnected -ne $true){
 		send-SMTPmail -to $EmailTo -from $EmailFrom -subject "ERROR: $VISRV Daily Report" -smtpserver $SMTPSRV -body "The Connect-VISERVER Cmdlet did not work, please check you VI Server."
 		exit
 	}
 	
 }
 #
 #.Description
 #.Parameter self
 #.Parameter credential
 #.Parameter url
 #.Parameter toFile
 #.Parameter bytes
     $webclient = New-Object Net.Webclient
     if ($credential) {
         $webClient.Credential = $credential
     }
     if ($self) {
         $webClient.UseDefaultCredentials = $true
     }
     if ($toFile) {
         if (-not "$toFile".Contains(":")) {
             $toFile = Join-Path $pwd $toFile
         }
         $webClient.DownloadFile($url, $toFile)
     } else {
         if ($bytes) {
             $webClient.DownloadData($url)
         } else {
             $webClient.DownloadString($url)
         }
     }
 $testok = Select-String -pattern $testpattern -Path $toFile
 if ( $testok -eq $null ) { 
 	Write-host "$testpattern not found at $url"
 	$FullVM = Get-View -ViewType VirtualMachine -Filter @{"Name" = $CHECKVM}
 	$NoTools = $FullVM | Where {$_.Runtime.Powerstate -eq �poweredOn� -And ($_.Guest.toolsStatus -eq �toolsNotInstalled� -Or $_.Guest.ToolsStatus -eq �toolsNotRunning�) }  | Select Name, @{N=�Status�; E={$_.Guest.ToolsStatus}}
 	if ( $NoTools ) {
 		Write-host "VMtools not running in $CHECKVM so forcing reboot using power switch."
 		Stop-VM $CHECKVM -Confirm:$false
 		Start-VM $CHECKVM
 		send-SMTPmail -to $EmailTo -from $EmailFrom -subject "Warning: $CHECKVM Healthcheck Report" -smtpserver $SMTPSRV -body "The webserv-healthcheck PowerShell script did not find $testpattern at $url.<br /><br />VMtools was not running so $CHECKVM brutally rebooted. Please check the website to ensure all is well."
 		} else {
 		Write-host "VMtools OK so gracefully restarting $CHECKVM."
 		Restart-VMGuest $CHECKVM
 		send-SMTPmail -to $EmailTo -from $EmailFrom -subject "Warning: $CHECKVM Healthcheck Report" -smtpserver $SMTPSRV -body "The webserv-healthcheck PowerShell script did not find $testpattern at $url.<br /><br />VMtools was running so $CHECKVM restarted gracefully. Please check the website to ensure all is well."
 		}
 	} 
 	else { Write-host "$testpattern present at $url" }
 	
`

