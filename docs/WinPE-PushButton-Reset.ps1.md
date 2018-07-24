---
Author: billamoore
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5165
Published Date: 2014-05-18t05
Archived Date: 2014-05-20t23
---

# winpe pushbutton reset - 

## Description

push button reset script for configuration manager task sequences.

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
 	WinPE Push Button Reset
 .DESCRIPTION
 	This script will create a push button reset environment for use with a Windows 8.x build using a Configuration 
 	Manager Task Sequence. The process for building the environment needed to run this script sucessfully is documented
         on my blog at http://www.billamoore.com/2014/03/13/powershell-automating-push-button-reset-configuration-manager/
 .NOTES
 	There is currently a Connect Bug (ID875599) filed for the -CompressionType parameter which is presently excluded from
         the New-WindowsImage command below. For now, no compression of the created wim is done. A DISM command could be substituted
         where compression is a concern. (It is for me.) PBRExclusions.ini is simply a DISM ini which excludes certain directories.
         Exclusion of _SMSTaskSequence would keep the wim clean for restore. 
 
         Because this image is not generalized it is not supported by Microsoft.
 
 .AUTHOR
 	Bill A Moore (billamoore@gmail.com)
 	Version 1.0, May 18, 2014
 	Prerequisites: PowerShell v4.0
 #>
     Set-Partition -DiskNumber 0 -PartitionNumber 1 -NewDriveLetter T
     Set-Partition -DiskNumber 0 -PartitionNumber 2 -NewDriveLetter S
     Set-Partition -DiskNumber 0 -PartitionNumber 4 -NewDriveLetter W
     New-Partition -DiskNumber 0 -UseMaximumSize -DriveLetter R | Format-Volume -FileSystem NTFS -NewFileSystemLabel "Recovery Image" -Confirm:$false
     New-Item R:\RecoveryImage -Type directory
     New-WindowsImage -ImagePath r:\RecoveryImage\install.wim -CapturePath W:\ -Name "Recovery Image" -ConfigFilePath X:\SMS\PKG\SMS10000\PBRExclusions.ini -NoRpFix
     
     Copy-Item -Recurse -Path R:\RecoveryImage -Destination W:\
     $Size = ( Get-ChildItem R:\RecoveryImage\ -Recurse -Force | Measure-Object -Property Length -Sum ).Sum + 500MB
     Remove-Partition -DriveLetter R -Confirm:$false
     $MaxSize = ( Get-PartitionSupportedSize -DriveLetter W ).sizeMax
     Resize-Partition -DriveLetter W -Size ( $MaxSize - $Size )
     New-Partition -DiskNumber 0 -UseMaximumSize -DriveLetter R -GptType '{de94bba4-06d1-4d40-a16a-bfd50179d6ac}' | Format-Volume -FileSystem NTFS -NewFileSystemLabel "Recovery Image" -Confirm:$false
     Move-Item -Path W:\RecoveryImage R:\
     CMD /c "W:\Windows\System32\icacls R:\RecoveryImage /inheritance:r /T"
     CMD /c "W:\Windows\System32\icacls R:\RecoveryImage /grant:r SYSTEM:(F) /T"
     CMD /c "W:\Windows\System32\icacls R:\RecoveryImage /grant:r *S-1-5-32-544:(F) /T"
     CMD /c "W:\Windows\System32\reagentc /setosimage /path R:\RecoveryImage /target W:\Windows /index 1"
     Copy-Item -Path X:\Windows\Temp\SMSTSLog\smsts.log -Destination W:\Logs\PEsmsts.log
     Copy-Item -Path X:\Windows\Logs\DISM\dism.log -Destination W:\Logs\DISM.log
`

