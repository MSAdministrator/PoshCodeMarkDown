---
Author: monahancj
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3923
Published Date: 2013-01-30t14
Archived Date: 2016-03-03t19
---

# ucs_faultreport.ps1 - 

## Description

basic cisco ucs fault report

## Comments



## Usage



## TODO



## function

`get-now`

## Code

`#
 #
 <#
 .Synopsis
     Connects to multiple UCS environments listed in a text file and generates a basic fault report for each one, then sends one email with the results.  In the report are unacknowledged faults, hardware locator LEDs that are on, and if you are short on port licenses it will show that.  Variables and paths will have to be changed to match your environment.
 .Blog
 	batchlife.wordpress.com
 #>
 
 param($CredentialFile=$null)
 
 if ($CredentialFile -eq $null) { Write-Host "Missing the path to the credential file." -BackgroundColor DarkYellow -ForegroundColor DarkRed; break }
 
 
 #-----------------------------
 #-----------------------------
 function Get-Now { (get-date -uformat %Y%m%d) + "_" + (get-date -uformat %H%M%S) }
 if ((Get-Module -Name CiscoUcsPS) -eq $null) { Import-Module -Name C:\Ops\Modules\CiscoUCSPowerTool\CiscoUcsPS.psd1 }
 
 Set-Variable -Name ScriptDir   -Value "C:\Scripts" -Scope Local
 Set-Variable -Name ReportPath  -Value "C:\Scripts\Reports\UCS_Fault_Reports" -Scope Local
 Set-Variable -Name ReportFile  -Value "$($ReportPath)\UCS_Fault_Report_$(Get-Now).html" -Scope Local
 Write-Verbose "`nStatic parameters`n-----------------`nScriptDir = $($ScriptDir)`nReportFile = $($ReportFile)`n"
 $mailTo = "alertsdistgroup@yourcompany.com"
 $mailFrom = "UCS_Fault_Reports@yourcompany.com"
 $mailSMTP = "smtp.yourcompany.com"
 
 If ( !(Test-Path $ReportPath) ) { mkdir $ReportPath }
 . $ScriptDir\Hals_PSCredentials.ps1
 
 $UcsCred = Import-PsCredential -Path $CredentialFile
 
 
 #-----------------------------
 #-----------------------------
 
 
 "<pre>" | Out-File -FilePath $ReportFile -Width 400
 Get-Date | Out-File -FilePath $ReportFile -Append -Width 400
 
 Get-Content C:\Scripts\Repo\StaticInfo\UCSs.txt | ForEach-Object { 
 	Write-Output "=========================================================================================================="
 	Connect-Ucs -Name $_ -Credential $UcsCred | select name,version,username,virtualipv4address | ft -a
 	if ($DefaultUcs -eq $null) {
 		$mailBody = @()
     	$mailBody += "$(Get-Now) - Error connecting to UCS`n`n"
     	$mailBody += $_
     	Send-MailMessage  -From $mailFrom -To $mailTo -Subject "Backup-mUcs Error-- Failed connecting to $($Ucs)" -Body ($mailBody | Out-String) -SmtpServer $mailSMTP
 	}
 	else {
 		Get-UcsFault | Where-Object { ($_.Severity -ne 'cleared') -and ($_.Ack -ne 'yes') } | sort lasttransition -Descending | select lasttransition,severity,status,type,dn,descr | ft -a
 		Get-UcsLocatorLed | ? { $_.operstate -eq 'on' } | sort dn | select dn,adminstate,color,operstate,id | ft -a
 		If (Get-UcsFault | Where-Object { ($_.Severity -ne 'cleared') -and ($_.Dn -match 'F0676') } ) { 
 			"`n------ Port Licenses Needed ----------------------------------------------------------------------------"
 			Get-UcsLicense | Select-Object Ucs,Scope,Feature,Sku,AbsQuant,DefQuant,UsedQuant,@{n="RemQuant";e={$_.AbsQuant-$_.UsedQuant}},Status,PeerStatus,OperState,GracePeriodUsed | ft -a
 			Get-UcsLicenseServerHostId | select ucs,rn,hostid | ft -a
 		}
 		Disconnect-Ucs 
 	}
 } | Out-File -FilePath $ReportFile -Append -Width 400
 
 "</pre>" | Out-File -FilePath $ReportFile -Append -Width 400
 
 $body = Get-Content $ReportFile
 
 Send-MailMessage  -From $mailFrom -To $mailTo -Subject "UCS Fault Report $(Get-Date)" -Body ($body | Out-String) -BodyAsHTML -SmtpServer $mailSMTP
`

