---
Author: colin squier
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6218
Published Date: 2017-02-15t21
Archived Date: 2017-05-22t04
---

# enable bitlocker - 

## Description

enables bitlocker on the os drive. if bitlocker has already been enabled, extracts the recovery key if there is one present.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $BitlockerStatus = (Get-BitLockerVolume -MountPoint $env:SystemDrive).ProtectionStatus
 $RecoveryKeyPath = "\\<servername>\USER\users\<username>"
 $RecoveryKeyFilePath = "Z:"
 
 if ($BitlockerStatus -eq "On")
 {
     Write-Host "BitLocker already enabled on $env:SystemDrive"
     $BitLocker = ((Get-BitLockerVolume -MountPoint $env:SystemDrive).KeyProtector | Where-Object { $_.KeyProtectorType -eq 'RecoveryPassword' })
     if ($BitLocker.KeyProtectorType -eq "RecoveryPassword")
     {
         $BitLockerId = $Bitlocker.KeyProtectorId
         $BitlockerId = ($BitLockerId -replace '[{}]','')
         $RecoveryKeyFileName = "BitLocker Recovery Key " + $BitLockerId + " " + $env:COMPUTERNAME + ".txt"
         $RecoveryKeyDrive = (Get-PSDrive -PSProvider FileSystem)
         if (($RecoveryKeyDrive | Where-Object { $_.Name -eq "Z" }).Name -ne "Z")
         {
             New-PSDrive -Name "Z" -PSProvider FileSystem -Root $RecoveryKeyPath
         }
 
         if (!(Test-Path $RecoveryKeyFilename))
         {
             New-Item -Path Z: -ItemType File -Name $RecoveryKeyFileName
         }
         $RecoveryKey = $BitLocker.RecoveryPassword
         $RecoveryFileContent = @"
 BitLocker Drive Encryption recovery key 
 
 To verify that this is the correct recovery key, compare the start of the following identifier with the identifier value displayed on your PC.
 
 Identifier:
 
     $BitLockerId
 
 If the above identifier matches the one displayed by your PC, then use the following key to unlock your drive.
 
 Recovery Key:
 
     $RecoveryKey
 
 If the above identifier doesn't match the one displayed by your PC, then this isn't the right key to unlock your drive.
 Try another recovery key, or refer to http://go.microsoft.com/fwlink/?LinkID=260589 for additional assistance.
 "@
         $RecoveryKeyFileName = (Join-Path Z: -ChildPath $RecoveryKeyFileName)
         $RecoveryFileContent | Out-File $RecoveryKeyFileName -Encoding UTF8
         (Get-Content $RecoveryKeyFileName | Out-String) -replace "`n", "`r`n" | Out-File $RecoveryKeyFileName -Encoding UTF8
         if (($RecoveryKeyDrive | Where-Object { $_.Name -eq "Z" }).Name -eq "Z")
         {
             Remove-PSDrive -Name "Z"
         }
     }
 }
 else
 {
     Write-Host "Enabling BitLocker encryption on $env:SystemDrive, this will take some time."
     $RecoveryKeyDrive = (Get-PSDrive -PSProvider FileSystem)
     if (($RecoveryKeyDrive | Where-Object { $_.Name -eq "Z" }).Name -ne "Z")
     {
         New-PSDrive -Name "Z" -PSProvider FileSystem -Root $RecoveryKeyPath
     }
     Enable-BitLocker -EncryptionMethod Aes128 -RecoveryKeyPath $RecoveryKeyFilePath -RecoveryKeyProtector -MountPoint $env:SystemDrive -SkipHardwareTest
 
     $BitLocker = ((Get-BitLockerVolume -MountPoint $env:SystemDrive).KeyProtector | Where-Object { $_.KeyProtectorType -eq 'RecoveryPassword' })
     $BitLockerId = $Bitlocker.KeyProtectorId
     $BitlockerId = ($BitLockerId -replace '[{}]','')
     $RecoveryKeyFileName = "BitLocker Recovery Key " + $BitLockerId + ".txt"
     $File = (Join-Path $RecoveryKeyFilePath -ChildPath $RecoveryKeyFileName)
 
     if(Test-Path $File)
     {
         $RecoveryKeyFileName = "Bitlocker Recovery Key " + $BitLockerId + " " + $env:COMPUTERNAME + ".txt"
         Rename-Item -Path $File -NewName $RecoveryKeyFileName
         [System.IO.FileInfo]$RecoveryKeyFile = (Get-ChildItem -Path $RecoveryKeyFilePath -Name $RecoveryKeyFileName)
     }
 
     Do
     {
         Start-Sleep -Seconds 30
         Get-BitLockerVolume | Select MountPoint,VolumeStatus,EncryptionPercentage,ProtectionStatus
     } While (((Get-BitLockerVolume).EncryptionPercentage) -lt 100)
 
     $BitLocker = ((Get-BitLockerVolume -MountPoint $env:SystemDrive).KeyProtector | Where-Object { $_.KeyProtectorType -eq 'Tpm' })
     if ($BitLocker.KeyProtectorType -eq "Tpm")
     {
         Get-BitLockerVolume | Add-BitLockerKeyProtector -TpmProtector
     }
 
     if (($RecoveryKeyDrive | Where-Object { $_.Name -eq "Z" }).Name -eq "Z")
     {
         Remove-PSDrive -Name "Z"
     }
 }
`

