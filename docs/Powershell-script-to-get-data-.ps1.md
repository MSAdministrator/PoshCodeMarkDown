---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4113
Published Date: 
Archived Date: 2013-05-09t16
---

#  - 

## Description

powershell script to get data from exchange server

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $WarningPreference = "SilentlyContinue"
 $password = Get-Content C:\securestring.txt | convertto-securestring
 $username = "PROD\administrator"
 $credentials = new-object -typename System.Management.Automation.PSCredential -argumentlist $username, $password
 
 $uucenter = "uuoresund.dk"
 $totalSize = 0
 
 $session = New-PSSession -ConfigurationName Microsoft.Exchange -ConnectionUri https://prod-exch1.prod.local/PowerShell/ -Credential $credentials
 Import-PSSession $session -DisableNameChecking | out-null
 
 foreach ($mbx in Get-Mailbox) 
 {
     if($mbx.PrimarySMTPAddress -like '*uuoresund.dk*')
     {
         $size = (Get-MailboxStatistics $mbx.Identity).TotalItemSize
 
         $position = $size.IndexOf("(")
         $size = $size.Substring($position+1)
 
         $position = $size.IndexOf(" ")
         $size = $size.Substring(0, $position)
 
         $totalSize = [long]$totalSize + [long]$size
     }
 }
 
 $totalSize = $totalSize/1000000
 
 Write-Host "<prtg>"
 Write-Host "    <result>"
 Write-Host "        <channel>$uucenter</channel>"
 Write-Host "        <value>$totalSize</value>"
 Write-Host "        <float>1</float>"
 Write-Host "    </result>"
 Write-Host "</prtg>"
 
 Remove-PSSession $session
 
 Exit 0
`

