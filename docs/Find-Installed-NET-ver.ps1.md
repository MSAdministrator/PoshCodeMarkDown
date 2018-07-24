---
Author: joakim
Publisher: 
Copyright: 
Email: 
Version: 3.5
Encoding: ascii
License: cc0
PoshCode ID: 3297
Published Date: 2012-03-25t13
Archived Date: 2012-03-28t09
---

# find installed .net ver - 

## Description

a script for finding installed .net versions on remote workstations or servers. see the full documentation at powershelladmin.com ( http

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 param([Parameter(Mandatory=$true)][string[]] $ComputerName,
       [switch] $Clobber)
 
 
 function ql { $args }
 
 function Quote-And-Comma-Join {
     
     param([Parameter(Mandatory=$true)][string[]] $Strings)
     
     $Strings = $Strings | ForEach-Object { $_ -replace '[\r\n]', '' }
     ($Strings | ForEach-Object { '"' + ($_ -replace '"', "'") + '"' }) -join ','
     
 }
 
 
 Set-StrictMode -Version Latest
 $ErrorActionPreference = 'Stop'
 
 $StartTime = Get-Date
 "Script start time: $StartTime"
 
 $Date = (Get-Date).ToString('yyyy-MM-dd')
 $OutputOnlineFile  = ".\DotNetOnline-${date}.txt"
 $OutputOfflineFile = ".\DotNetOffline-${date}.txt"
 $CsvOutputFile = ".\DotNet-Versions-${date}.csv"
 
 if (-not $Clobber) {
     
     $FoundExistingLog = $false
     
     foreach ($File in $OutputOnlineFile, $OutputOfflineFile, $CsvOutputFile) {
         
         if (Test-Path -PathType Leaf -Path $File) {
             
             $FoundExistingLog = $true
             "$File already exists"
             
         }
     
     }
     
     if ($FoundExistingLog -eq $true) {
         
         $Answer = Read-Host "The above mentioned log file(s) exist. Overwrite? [yes]"
         
         if ($Answer -imatch '^n') { 'Aborted'; exit 1 }
         
     }
     
 }
 
 Remove-Item $OutputOnlineFile -ErrorAction SilentlyContinue
 Remove-Item $OutputOfflineFile -ErrorAction SilentlyContinue
 Remove-Item $CsvOutputFile -ErrorAction SilentlyContinue
 
 $Counter    = 0
 $DotNetData = @{}
 $DotNetVersionStrings = ql v4\Client v4\Full v3.5 v3.0 v2.0.50727 v1.1.4322
 $DotNetRegistryBase   = 'SOFTWARE\Microsoft\NET Framework Setup\NDP'
 
 foreach ($Computer in $ComputerName) {
     
     $Counter++
     $DotNetData.$Computer = New-Object PSObject
     
     if ($Computer -notmatch '^\S') {
         
         Write-Host -Fore Red "Skipping malformed item/line ${Counter}: '$Computer'"
         Add-Member -Name Error -Value "Malformed argument ${Counter}: '$Computer'" -MemberType NoteProperty -InputObject $DotNetData.$Computer
         continue
         
     }
     
     if (Test-Connection -Quiet -Count 1 $Computer) {
         
         Write-Host -Fore Green "$Computer is online. Trying to read registry."
         
         $Computer | Add-Content $OutputOnlineFile
         
         $ErrorActionPreference = 'SilentlyContinue'
         $Registry = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', $Computer)
         $RegSuccess = $?
         $ErrorActionPreference = 'Stop'
                 
         if ($RegSuccess) {
             
             Write-Host -Fore Green "Successfully connected to registry of ${Computer}. Trying to open keys."
             
             foreach ($VerString in $DotNetVersionStrings) {
                 
                 if ($RegKey = $Registry.OpenSubKey("$DotNetRegistryBase\$VerString")) {
                     
                     #"Successfully opened .NET registry key (SOFTWARE\Microsoft\NET Framework Setup\NDP\$verString)."
                     
                     if ($RegKey.GetValue('Install') -eq '1') {
                         
                         #"$computer has .NET $verString"
                         Add-Member -Name $VerString -Value 'Installed' -MemberType NoteProperty -InputObject $DotNetData.$Computer
                         
                     }
                     
                     else {
                         
                         Add-Member -Name $VerString -Value 'Not installed' -MemberType NoteProperty -InputObject $DotNetData.$Computer
                         
                     }
                     
                 }
                 
                 else {
                     
                     Add-Member -Name $VerString -Value 'Not installed (no key)' -MemberType NoteProperty -InputObject $DotNetData.$Computer
                     
                 }
                 
             }
             
         }
         
         else {
             
             Write-Host -Fore Yellow "${Computer}: Unable to open remote registry key."
             Add-Member -Name Error -Value "Unable to open remote registry: $($Error[0].ToString())" -MemberType NoteProperty -InputObject $DotNetData.$Computer
             
         }
         
     }
     
     else {
         
         Write-Host -Fore Yellow "${Computer} is offline."
         Add-Member -Name Error -Value "No ping reply" -MemberType NoteProperty -InputObject $DotNetData.$Computer
         $Computer | Add-Content $OutputOfflineFile
         
     }    
     
 }
 
 $CsvHeaders = @('Computer') + @($DotNetVersionStrings) + @('Error')
 $HeaderLine = Quote-And-Comma-Join $CsvHeaders
 Add-Content -Path $CsvOutputFile -Value $HeaderLine
 
 $DotNetData.GetEnumerator() | ForEach-Object {
     
     $Computer = $_.Name
     
     $TempData = @{}
     $TempData.'Computer' = $Computer
     
     if (Get-Member -InputObject $DotNetData.$Computer -MemberType NoteProperty -Name Error) {
         
         $TempData.'Error' = $DotNetData.$Computer.Error
         
         foreach ($VerString in $DotNetVersionStrings) {
             
             $TempData.$VerString = 'Unknown'
             
         }
         
         
     }
     
     else {
         
         $TempData.'Error' = '-'
         
         foreach ($VerString in $DotNetVersionStrings) {
             
             $TempData.$VerString = $DotNetData.$Computer.$VerString
             
         } 
         
     }
     
     
     $TempArray = @()
     
     foreach ($Header in $CsvHeaders) {
         
         $TempArray += $TempData.$Header
         
     }
     
     $CsvLine = Quote-And-Comma-Join $TempArray
     Add-Content -Path $CsvOutputFile -Value $CsvLine
     
 }
 
 @"
 Script start time: $StartTime
 Script end time:   $(Get-Date)
 Output files: $CsvOutputFile, $OutputOnlineFile, $OutputOfflineFile
 "@
`

