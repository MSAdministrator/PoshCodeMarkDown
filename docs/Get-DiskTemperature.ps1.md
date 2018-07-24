---
Author: akaneo
Publisher: 
Copyright: 
Email: 
Version: 1.00
Encoding: ascii
License: cc0
PoshCode ID: 4364
Published Date: 2013-08-05t14
Archived Date: 2016-11-03t05
---

# get-disktemperature - 

## Description

uses get-storagereliabilitycounter method to obtain information about the temperature of the physical disks

## Comments



## Usage



## TODO



## function

`get-disktemperature`

## Code

`#
 #
 Function Get-DiskTemperature {
 <#
 .Synopsis
 Shows current temperature of hard drives
 .Description
 Uses Get-StorageReliabilityCounter method to obtain information about the temperature of the physical disks
 .Example
 PS C:\> Get-DiskTemperature -DiskNumbers 0,1 -TechInfo | Format-Table -AutoSize
 Caption                Number DriveLetters Temperature ConvenientSize BusType FirmwareVersion PowerOnHours
 -------                ------ ------------ ----------- -------------- ------- --------------- ------------
 OCZ-VERTEX4                 0 {R, C}                   238.5 GiB      SATA    1.5                     6175
 ST1000LM024 HN-M101MBB      1 {L}          38          931.5 GiB      USB     2AR1                    1144
 .Link
 Get-Disk
 Get-Partition
 Get-StorageReliabilityCounter
 #>
 [CmdletBinding()]
 Param(
 [Parameter(Mandatory=$false,Position=1)]
 [System.Array]$DiskNumbers,
 [switch]$TechInfo
 )
 New-Variable -Name space -Value ([System.Char]32) -Option Const
 New-Variable -Name csz_MeasureLabel -Value @('Bytes', 'KiB', 'MiB', 'GiB', 'TiB', 'PiB') -Option Const
 filter Remove-ExtraSpaces {
 $rex_OutStr = [System.String]::Empty
 $rex_NoMoreSpaces = $true
 $rex_IsReadyForNextSpace = $false
 $_.GetEnumerator() | ForEach-Object -Process {
 if ($_ -ne $space)
 {
 if ($rex_IsReadyForNextSpace)
 {
 $rex_OutStr += $space
 $rex_IsReadyForNextSpace = $false
 }
 $rex_OutStr += $_
 $rex_NoMoreSpaces = $false
 }
 else
 {
 if (-not $rex_NoMoreSpaces)
 {
 $rex_IsReadyForNextSpace = $true
 $rex_NoMoreSpaces = $true
 }
 }
 }
 Return $rex_OutStr
 }
 filter Get-ConvenientSize {
 if (![bool]$_ -or $_ -lt 1 -or $_ -gt ([System.Int64]::MaxValue -shr 3))
 {
 return $null
 }
 else
 {
 $csz_MeasureValue = [System.Int64]1
 for ($k = 0; $k -le $csz_MeasureLabel.Count - 1; ++$k)
 {
 $csz_MeasureValue = [System.Int64]$csz_MeasureValue -shl 10
 if ($_ -lt $csz_MeasureValue)
 {
 $csz_Measure = $k
 $csz_MeasureValue = [System.Int64]$csz_MeasureValue -shr 10
 break
 }
 }
 $csz_MeasureRound = $csz_Measure - 2
 if ($csz_MeasureRound -lt 0) {$csz_MeasureRound = 0}
 $csz_ConvenientSize = [System.Math]::Round($_ / $csz_MeasureValue, $csz_MeasureRound)
 return [System.String]$csz_ConvenientSize + $space + $csz_MeasureLabel[$csz_Measure]
 }
 }
 switch ($DiskNumbers)
 {
 $null {$gdt_Drives = Get-Disk | Sort-Object -Property 'Number'}
 default {$gdt_Drives = Get-Disk -Number $DiskNumbers | Sort-Object -Property 'Number'}
 }
 $gdt_PartitionsInfo = Get-Partition | Select-Object -Property 'DriveLetter','DiskNumber'
 $gdt_Drives | ForEach-Object -Process {
 $fe_Letters = @()
 $fe_Number = $_.Number
 $gdt_PartitionsInfo | ForEach-Object -Process {
 if ($_.DiskNumber -eq $fe_Number -and $_.DriveLetter -ne [System.Char]0)
 {
 $fe_Letters += $_.DriveLetter
 }
 }
 if (![bool]$fe_Letters.Count)
 {
 $fe_Letters = $null
 }
 $fe_SRC = $_ | Get-StorageReliabilityCounter | Select-Object -Property 'Temperature','PowerOnHours'
 $fe_Caption = $_.Manufacturer + $_.Model | Remove-ExtraSpaces
 $_ | Add-Member -MemberType 'NoteProperty' -Name 'Caption' -Value $fe_Caption
 $_ | Add-Member -MemberType 'NoteProperty' -Name 'DriveLetters' -Value $fe_Letters
 $_ | Add-Member -MemberType 'NoteProperty' -Name 'Temperature' -Value $fe_SRC.Temperature
 if ($TechInfo)
 {
 $fe_ConvenientSize = $_.Size | Get-ConvenientSize
 $_ | Add-Member -MemberType 'NoteProperty' -Name 'ConvenientSize' -Value $fe_ConvenientSize
 $_ | Add-Member -MemberType 'NoteProperty' -Name 'PowerOnHours' -Value $fe_SRC.PowerOnHours
 }
 }
 switch ($TechInfo)
 {
 $false {
 Write-Output $gdt_Drives | Select-Object -Property 'Caption','Number','DriveLetters','Temperature'
 }
 $true {
 Write-Output $gdt_Drives | Select-Object -Property 'Caption','Number','DriveLetters','Temperature','ConvenientSize','BusType','FirmwareVersion','PowerOnHours'
 }
 }
`

