---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 4512
Published Date: 2015-10-08t17
Archived Date: 2015-05-18t02
---

# scom gw certificate ae - 

## Description

automation of scom gateway certificate renewal (you need to configure autoenrollment separetly)

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 
 
 $SCOMTemplateName="SCOM Template"
 
 $SCOMCertRegPath="HKLM:\SOFTWARE\Microsoft\Microsoft Operations Manager\3.0\Machine Settings"
 
 $SCOMCertRegValueName="ChannelCertificateSerialNumber"
 
 
 $ParsedCertificates=@()
 
 $LocalCertificates=Get-ChildItem Cert:\LocalMachine\My
 
  foreach ($LocalCertificate in $LocalCertificates) {
 
 	$ParsedCertificates+= $LocalCertificate | Select `
 		Friendlyname,
 		Thumbprint,
         SerialNumber,
         NotAfter,
         NotBefore,
 		@{Name="Template";Expression={($_.Extensions | 
 			Where-Object {$_.oid.Friendlyname -match "Certificate Template Information"}).Format(0) -replace "(.+)?=(.+)\((.+)?", '$2'}},
 		@{Name="Subject";Expression={$_.SubjectName.name}}
 }
 
 $SerialNumber=($ParsedCertificates | Where-Object { $_.Template -eq $SCOMTemplateName } | Sort-Object NotAfter -Descending | select -First 1).SerialNumber
 
 $ReversedPairs=[regex]::Matches($SerialNumber,'..','RightToLeft') | ForEach-Object { $_.Value }
 
 $ReversedPairsInBinary=$ReversedPairs | ForEach-Object { [convert]::ToByte($_,16) }
 
 $CurrentSCOMCertificate=Get-ItemProperty -Path $SCOMCertRegPath | Select-Object $SCOMCertRegValueName -ExpandProperty $SCOMCertRegValueName
 
 if (($ReversedPairsInBinary -join "") -eq ($CurrentSCOMCertificate -join "")) {
     Write-Output "The current certificate is the latest."
 }
 else {
     Write-Output "New certificate found. Changing registry..."
     New-ItemProperty -Path $SCOMCertRegPath -Name $SCOMCertRegValueName -Value $ReversedPairsInBinary -Type Binary -Force
 
     Write-Output "Restarting health service..."
     Restart-Service -Name HealthService -Force
 }
`

