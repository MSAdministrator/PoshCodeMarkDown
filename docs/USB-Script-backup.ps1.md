---
Author: archdeacon
Publisher: 
Copyright: 
Email: 
Version: 3.1
Encoding: ascii
License: cc0
PoshCode ID: 3176
Published Date: 2012-01-18t22
Archived Date: 2012-01-21t08
---

# usb script backup - 

## Description

backup any recently changed powershell scripts to a usb drive.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
 Backup-ToUSB.ps1 (Version 3.1, 19 Jan 2012)
 .DESCRIPTION 
 This script will backup recently changed *.ps1,*.psm1,*.psd1 files from any 
 selected folder (default is $pwd) to any number of inserted USB devices, on
 which an archive folder 'PSarchive' will be created if it does not already 
 exist. 
 Any USB Hard Disk will appear as a local system disk (DeviceType 3) so the
 default 'Exclude' pattern is set to drives C:\ and D:\ which can be changed
 to suit.
 .EXAMPLE
 Invoke-Expression ".\Backup-ToUsb.ps1 -exclude cd $args"
 Add this in a function in $profile and it will exclude drives C and D from 
 the backup devices used.
 .EXAMPLE
 .\Backup-ToUSB.ps1 ps* -exclude cde -v
 Backup all ps* files and exclude drives C,D,E and show Verbose output.
 .EXAMPLE
 .\Backup-ToUSB.ps1 -type ps1 -v
 Select *.ps1 files to be archived. 'Verbose' will display the name of each 
 file processed and the size (in Kb) on disk. Using the '-Debug' switch shows
 the time difference that has tagged any file for archive. 
 .NOTES
 If the message 'WARNING: Error on drive X:\ - restart.' keeps appearing, 
 manually enter 'Test-Path X:\' before resubmitting, which seems to cure it!
 The author may be contacted via www.SeaStarDevelopment.Bravehost.com  
 #>
 
 
 param ([String]$types = 'ps1',
        [String]$folder = $pwd,
        [String]$exclude  = 'cd',
        [Switch]$debug,
        [Switch]$verbose)
 if ($verbose) {
    $verbosePreference = 'Continue'
 }
 if ($debug) {
    $debugPreference = 'Continue'
 }
 $drive = '-<BLANK>-'
    Write-Warning "Invalid filetype ($types): ps1, psm1, psd1 only."
    exit 1
 } 
 if (!(Test-path $folder -pathType Container )) { 
    [System.Media.SystemSounds]::Hand.Play() 
    Write-Warning "$folder is not a directory - resubmit." 
    exit 3 
 } 
 function copyFile ([string]$value, [String]$value1) {
    Copy-Item -Path $value -Destination $newFolder -Force
    Write-Verbose "--> Archiving file: (Size $value1) $value" 
    $SCRIPT:files++  
 }
 $exclude = "[$exclude]:"
 Get-WMIObject Win32_LogicalDisk -filter 'DriveType = 2 OR DriveType = 3' | 
    Select-Object VolumeName, FreeSpace, DeviceID | 
       Where-Object {($_.VolumeName -ne $null) -and
                     ($_.DeviceID -notmatch $exclude)} |
 foreach {
     $vol   = $_.VolumeName
     $drive = $_.DeviceID + '\'
     [int]$free  = $_.FreeSpace / 1MB
     Write-Verbose "Scanning USB devices - Drive = [$drive] Name = $vol, Free = $free Mb"
     if (!(Test-Path $drive)) { 
        Write-Warning "Error on drive $drive - restart." 
        exit 4    
     } 
     if (!(Test-Path $newFolder)) { 
        New-Item -path ($drive) -name PSarchive -type directory | Out-Null 
     } 
     (Get-ChildItem $folder -filter "*.$types") |  
     foreach {
         $checkFile = Join-Path ($drive + 'PSArchive') $_
         $fsize = "{0:0000}" -f ($_.Length /1kb)
         if ($fsize -eq '0000') {
            $fsize = '<1kb'
         }
         if (Test-path $checkFile) {
            $lwtUsb = (Get-Item $checkFile).LastWriteTime
            if ($lwtHdd -gt $lwtUsb) {
               Write-Debug "(HDD) $lwtHdd   (USB) $LwtUsb  $_"
               copyFile $_ $fSize
            }
         }
            Write-Debug "(HDD) $($_.LastWriteTime)   (USB  Archiving new file)  $_"
            copyFile $_ $fsize
         }                           
     } 
     $print =  "Backup to USB drive [{0}] (Volume = {2}) completed; {1} files copied." -f $drive, $SCRIPT:files, $vol 
     Write-Host "--> $print"
 }
 if ($drive -eq '-<BLANK>-') { 
     [System.Media.SystemSounds]::Asterisk.Play() 
     Write-Warning "No USB drive detected - Insert a device and resubmit."   
 }
`

