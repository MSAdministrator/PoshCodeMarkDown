---
Author: _rov3
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6317
Published Date: 2016-04-22t20
Archived Date: 2016-04-24t06
---

# mail signature service - 

## Description

updated with improved error trapping and fixed time stamps in log file!

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 	trap { (get-date -Format g) + "Service stopped`: {0}" -f $_.Exception.Message | Out-File $configfile.settings.siglogerror -Append; continue  }
 	
 	$PSScriptRoot = Split-Path -Parent -Path $MyInvocation.MyCommand.Definition
 	Set-Location -Path "$PSScriptRoot"
 		
 	[xml]$configfile = Get-Content configuration.xml
 	
 	Import-Module $configfile.settings.modulepath
 	
 	
 	if ((Test-Path $configfile.settings.siglog) -eq $false) {
 		New-Item $configfile.settings.siglog -type file
 		}
 	
 	if ((Test-Path $configfile.settings.siglogerror) -eq $false) {
 		New-Item $configfile.settings.siglogerror -type file
 		}
 	
 	
 	if ((Test-Path $configfile.settings.tempdir) -eq $false) {
 		New-Item $configfile.settings.tempdir -type folder
 		}
 	
 	
 	if ((Test-Path $configfile.settings.outputdir) -eq $false) {
 		"Error`: Output path is invalid. Please check configuration.xml and restart service." | Out-File $configfile.settings.siglogerror -Append
 		$serviceStop | Out-File $configfile.settings.siglogerror -Append
 		break
 		}
 	
 	
 	if ((Test-Path $configfile.settings.modulepath) -eq $false) {
 		"Error`: createSignature.psm1 not found. Please check configuration.xml and restart service." | Out-File $configfile.settings.siglogerror -Append
 		$serviceStop | Out-File $configfile.settings.siglogerror -Append
 		break
 		}
 	
 	$sigLog = Get-Content -path $configfile.settings.siglog
 	
 		
 
 (get-date -Format g) + " Service has started." | Out-File $configfile.settings.siglogerror -Append
 
 
 while($true) {
 
 	try {
 		$getUser = Get-ADUser -Searchbase $configfile.settings.searchbase -Properties * -Filter {Enabled -eq $true -and Mail -like '*'} | Where {$sigLog -notcontains $_.SamAccountName}
 		(get-date -Format g) + " Signatures for " + $getUser.Count + " users to be created." | Out-File $configfile.settings.siglogerror -Append
 		}
 	catch {
 		(get-date -Format g) + " Searchbase error. Please check configuration.xml and restart service`: $_" | Out-File $configfile.settings.siglogerror -Append
 		break
 		}
 		
 	$getUser | foreach {
 			
 		$sigFilename = ($_.samaccountname)+".htm"
 		
 		
 		$sigtempFile = $configfile.settings.tempdir + "\" + $sigFilename
 		$copyDir = $configfile.settings.outputdir + "\" + $sigFilename
 		
 		createSignature -template $configfile.settings.template -Name $_.Name -Title $_.title -StreetAddress $_.streetAddress -City $_.City -postalCode $_.postalCode -OfficePhone $_.OfficePhone -MobilePhone $_.mobilePhone -Company "Paperchase Products Ltd" -Sigpath $sigtempFile
 		
 		Copy-Item $sigtempFile $copyDir -Force
 		Remove-Item $sigtempFile -Force
 		
 		if ((Test-Path $copyDir) -eq $true) {
 			($_.samaccountname) | Out-File $configfile.settings.siglog -Append
 			}
 		else {
 			(get-date -Format g) + "Error creating`:" + ($_.samaccountname) | Out-File $configfile.settings.siglogerror -Append
 			}
 		
 		}
 	
 	start-sleep -Seconds $configfile.settings.generationinterval
 }
`

