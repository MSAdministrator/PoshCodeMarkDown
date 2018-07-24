---
Author: battlechicken
Publisher: 
Copyright: 
Email: 
Version: 6.1
Encoding: ascii
License: cc0
PoshCode ID: 6522
Published Date: 2016-09-18t02
Archived Date: 2016-11-12t03
---

# ie11 install - 

## Description

i wrote this to install ie11 through altiris.  though the install method works fine without using altiris.  the script installs all of the kb prerequisite and recommended updates for ie 11, which are specified in $updatemsus for the x86 and x64 blocks of code.  it then installs ie 11 without forcing a reboot.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 Requisite updates for IE11 installation on Microsoft Windows 7 x86 and x64 (Please see http://support.microsoft.com/kb/2847882)
 
     1. KB2729094 (http://support.microsoft.com/kb/2729094)
     2. KB2731771 (http://support.microsoft.com/kb/2731771)
     3. KB2533623 (http://support.microsoft.com/kb/2533623)
     4. KB2670838 (http://support.microsoft.com/kb/2670838)
     5. KB2786081 (http://support.microsoft.com/kb/2786081)
     6. KB2834140 (http://support.microsoft.com/kb/2834140)
 
 Optional updates for IE11 installation on Microsoft Windows 7 x86 and x64
 
     1. KB2639308 (http://support.microsoft.com/kb/2639308)
     2. KB2888049 (http://support.microsoft.com/kb/2888049)
     3. KB2882822 (http://support.microsoft.com/kb/2882822)
 #>
 
 $targetIEVer = 11
 $x64IEPath = "C:\Program Files\Internet Explorer\iexplore.exe"
 $x86IEPath = "C:\Program Files (x86)\Internet Explorer\iexplore.exe"
 if (([int](get-item $x64IEPath).VersionInfo.productVersion.split('.')[0] -ge $targetIEVer) -or ([int](get-item $x86IEPath).VersionInfo.productVersion.split('.')[0] -ge $targetIEVer)){
     Write-Host "IE11 Already installed - no action taken"
 }
 else{
 
     $os = (Get-WmiObject Win32_OperatingSystem -computername localhost).OSArchitecture
     try {
         if ($os -eq "32-Bit" -and (Test-Path $fileroot) -eq $true)
         {
 
             Write-Host "Found 32-Bit Architecture...Installing pre-requisite updates..."
 
             $updateMSU = @(
                 "Windows6.1-KB2533623-x86.msu",
                 "Windows6.1-KB2888049-x86.msu",
                 "Windows6.1-KB2670838-x86.msu",
                 "Windows6.1-KB2729094-v2-x86.msu",
                 "Windows6.1-KB2731771-x86.msu",
                 "Windows6.1-KB2786081-x86.msu",
                 "Windows6.1-KB2834140-v2-x86.msu",
                 "Windows6.1-KB2882822-x86.msu",
                 "Windows6.1-KB2639308-x86.msu"
             )
 
             foreach ($updateMSU in $updateMSUs){
                 Write-Host "Installing Update $updateMSU..."
                 Start-Process "wusa.exe" -ArgumentList @("""$fileroot\$updateMSU""","/quiet","/norestart") -Wait -Verbose
             }
 
             Write-Host "Stopping any active IE processes..."
             get-process iexplore -ErrorAction silentlycontinue | Stop-Process -ErrorAction SilentlyContinue
             Write-Host "Installing Internet Explorer 11, 32-bit...Please Wait"
             Start-Process "$fileroot\$ie32" -ArgumentList @("/passive","/update-no","/norestart") -Wait -verbose
 
             Exit 0
 
         }
     }
     catch {
         throw "Error installing IE 11"
     }
 
     #######################################################################################
     #######################################################################################
     try{
         if ($os -eq "64-Bit" -and (Test-Path $fileroot) -eq $true)
         {
             Write-Host "Found 64-Bit Architecture...Installing pre-requisite updates..."
 
             $updateMSUs = @(
                 "Windows6.1-KB2533623-x64.msu",
                 "Windows6.1-KB2888049-x64.msu",
                 "Windows6.1-KB2670838-x64.msu",
                 "Windows6.1-KB2729094-v2-x64.msu",
                 "Windows6.1-KB2786081-x64.msu",
                 "Windows6.1-KB2834140-v2-x64.msu",
                 "Windows6.1-KB2882822-x64.msu",
                 "Windows6.1-KB2639308-x64.msu"
             )
 
             foreach ($updateMSU in $updateMSUs){
                 Write-Host "Installing Update $updateMSU..."
                 Start-Process "wusa.exe" -ArgumentList @("""$fileroot\$updateMSU""","/quiet","/norestart") -Wait -Verbose
             }
 
 
             Write-Host "Stopping any active IE processes..."
             get-process iexplore -ErrorAction silentlycontinue | Stop-Process -ErrorAction SilentlyContinue
             Write-Host "Installing Internet Explorer 11, 64-Bit, Please wait..."
             Start-Process "$fileroot\$ie64" -ArgumentList @("/passive","/update-no","/norestart") -Wait -verbose
             if ($?){Write-Host "Install complete"}
             Exit 0
         }
     }
     catch {
             throw "Error installing IE 11"
     }
 }
`

