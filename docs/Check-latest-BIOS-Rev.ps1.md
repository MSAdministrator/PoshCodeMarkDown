---
Author: jeff patton
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2679
Published Date: 2012-05-17t09
Archived Date: 2016-03-16t14
---

# check latest bios rev - 

## Description

this script is pretty simple, it connects to a remote computer and grabs the bios class. it then connects to the dell support page for the remote computer’s servicetag. if the computer is a dell, it grabs the bios revision listed on the page. the inspiration came from reading the scripting guy blog about comments. (http

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $BiosRev = Get-WmiObject -Class Win32_BIOS -ComputerName $ComputerName -Credential $Credentials
 
 
 $DellBIOSPage = "http://support.dell.com/support/downloads/download.aspx?c=us&cs=RC956904&l=en&s=hied&releaseid=R294848&SystemID=PLX_960&servicetag=$($BiosRev.SerialNumber)&fileid=441102"
 
 
 $DellPageVersionString = "<span id=`"Version`" class=`"para`">"
 
 If ($BiosRev.Manufacturer -match "Dell")
 {
     $DellPage = (New-Object -TypeName net.webclient).DownloadString($DellBIOSPage)
     
     
     $DellCurrentBios = $DellPage.Substring($DellPage.IndexOf($DellPageVersionString)+$DellPageVersionString.Length,3)
 
     If (($BiosRev.SMBIOSBIOSVersion -eq $DellCurrentBios) -eq $false)
     {
         
         
         $BIOSDownloadURL = $DellPage.Substring($DellPage.IndexOf("http://ftp"),(($DellPage.Substring($DellPage.IndexOf("'http://ftp"),100)).indexof(".EXE"))+3)
         
         
         $BIOSFile = $BIOSDownloadURL.Substring(($BIOSDownloadURL.Length)-12,12)
 
         If ((Test-Path "C:\Dell\") -eq $false)
         {
             New-Item -Path "C:\" -Name "Dell" -ItemType Directory
         }
         If ((Test-Path "C:\Dell\$($ComputerName)") -eq $false)
         {
             New-Item -Path "C:\Dell" -Name $ComputerName -ItemType Directory
         }
 
         (New-Object -TypeName net.webclient).DownloadFile($BIOSDownloadURL,"C:\Dell\$($ComputerName)\$($BIOSFile)")
 
         Write-Host "Latest BIOS for $($ComputerName) downloaded to C:\Dell\$($ComputerName)\$($BIOSFile)"
     }
 }
`

