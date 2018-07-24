---
Author: huajun gu
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3623
Published Date: 2012-09-05t19
Archived Date: 2016-09-13t22
---

# new-xvm - 

## Description

this is a script which i have used to prepare my lab environment with hyper-v 3.0. i have updated the script after upgrading my test machines to windows server 2012 rtm last night. (9/5/2012)

## Comments



## Usage



## TODO



## function

`new-xvm`

## Code

`#
 #
 <#
 New-XVM -ComputerName HYPERVSVR02 -Name "WS2012-TESTSVR01" -SwitchName "External(192.168.1.0/24)" -VhdType NoVHD
 New-XVM -ComputerName HYPERVSVR02 -Name "WS2012-TESTSVR02" -SwitchName "External(192.168.1.0/24)" -VhdType ExistingVHD -VhdPath D:\vhds\WS2012-TESTSVR02.vhdx
 New-XVM -ComputerName HYPERVSVR02 -Name "WS2012-TESTSVR03" -SwitchName "External(192.168.1.0/24)" -VhdType NewVHD
 New-XVM -ComputerName HYPERVSVR02 -Name "WS2012-TESTSVR04" -SwitchName "External(192.168.1.0/24)" -VhdType NewVHD -DiskType Dynamic
 New-XVM -ComputerName HYPERVSVR02 -Name "WS2012-TESTSVR05" -SwitchName "External(192.168.1.0/24)" -VhdType NewVHD -DiskType Fixed -DiskSize 1GB
 New-XVM -ComputerName HYPERVSVR02 -Name "WS2012-TESTSVR06" -SwitchName "External(192.168.1.0/24)" -VhdType Differencing -ParentVhdPath 'D:\vhds\Windows Server 2012 RC Base.vhdx'
 New-XVM -ComputerName HYPERVSVR02 -Name "WS2012-TESTSVR07" -SwitchName "External(192.168.1.0/24)" -VhdType NewVHD -Configuration @{"MemoryStartupBytes"=1GB}
 #>
 
 Function New-XVM
 {
     [cmdletbinding()]
     Param
     (
         [Parameter(Mandatory=$false,Position=1)]
         [string]$ComputerName=$env:COMPUTERNAME,        
         [Parameter(Mandatory=$true,Position=2)]
         [string]$Name,
         [Parameter(Mandatory=$true,Position=3)]
         [string]$SwitchName,
         [Parameter(Mandatory=$true,Position=4)]
         [ValidateSet("NoVHD","ExistingVHD","NewVHD","Differencing")]
         [string]$VhdType,
         [Parameter(Mandatory=$false,Position=5)]
         [hashtable]$Configuration
     )
     DynamicParam
     {
         Switch ($VhdType) {
             "ExistingVHD" {
                 $attributes = New-Object System.Management.Automation.ParameterAttribute
                 $attributes.ParameterSetName = "ByParam"
                 $attributes.Mandatory = $true
                 $attributeCollection = New-Object -Type System.Collections.ObjectModel.Collection[System.Attribute]
                 $attributeCollection.Add($attributes)
                 $vhdPath = New-Object -Type System.Management.Automation.RuntimeDefinedParameter("VhdPath", [String], $attributeCollection)
                 $paramDictionary = New-Object -Type System.Management.Automation.RuntimeDefinedParameterDictionary
                 $paramDictionary.Add("VhdPath",$vhdPath)
                 return $paramDictionary
             }
             "NewVHD" {
                 $attributes = New-Object System.Management.Automation.ParameterAttribute
                 $attributes.ParameterSetName = "ByParam"
                 $attributes.Mandatory = $false
                 $attributeCollection = New-Object -Type System.Collections.ObjectModel.Collection[System.Attribute]
                 $attributeCollection.Add($attributes)
                 $diskType = New-Object -Type System.Management.Automation.RuntimeDefinedParameter("DiskType", [String], $attributeCollection)
                 $paramDictionary = New-Object -Type System.Management.Automation.RuntimeDefinedParameterDictionary
                 $paramDictionary.Add("DiskType",$diskType)
                 $attributes = New-Object System.Management.Automation.ParameterAttribute
                 $attributes.ParameterSetName = "ByParam"
                 $attributes.Mandatory = $false
                 $attributeCollection = New-Object -Type System.Collections.ObjectModel.Collection[System.Attribute]
                 $attributeCollection.Add($attributes)
                 $diskSize = New-Object -Type System.Management.Automation.RuntimeDefinedParameter("DiskSize", [uint64], $attributeCollection)
                 $paramDictionary.Add("DiskSize",$diskSize)
                 return $paramDictionary
             }
             "Differencing" {
                 $attributes = New-Object System.Management.Automation.ParameterAttribute
                 $attributes.ParameterSetName = "ByParam"
                 $attributes.Mandatory = $true
                 $attributeCollection = New-Object -Type System.Collections.ObjectModel.Collection[System.Attribute]
                 $attributeCollection.Add($attributes)
                 $parentVhdPath = New-Object -Type System.Management.Automation.RuntimeDefinedParameter("ParentVhdPath", [String], $attributeCollection)
                 $paramDictionary = New-Object -Type System.Management.Automation.RuntimeDefinedParameterDictionary
                 $paramDictionary.Add("ParentVhdPath",$parentVhdPath)
                 return $paramDictionary
             }
         }
     }
     Begin
     {
         Try
         {
             $vmHost = Get-VMHost -ComputerName $ComputerName -ErrorAction:Stop
         }
         Catch
         {
             $PSCmdlet.ThrowTerminatingError($Error[0])
         }
         $defaultVirtualHardDiskPath = $vmHost.VirtualHardDiskPath
     }
     Process
     {
         $validConfigNames = "MemoryStartupBytes","BootDevice"
         $configParams = @()
         Switch ($VhdType) {
             "NoVHD" {
                 $vhdFilePath = $null
             }
             "ExistingVHD" {
                 $vhdFilePath = $vhdPath.Value
             }
             "NewVhd" {
                 if (-not $diskType.IsSet) {$diskType.Value = "Dynamic"}
                 if (-not $diskSize.IsSet) {$diskSize.Value = 127GB}
                 $newVhdPath = Join-Path -Path $defaultVirtualHardDiskPath -ChildPath "$Name.vhdx"
                 Switch ($diskType.Value) {
                     "Fixed" {
                         $vhdFile = New-VHD -ComputerName $ComputerName -Fixed -SizeBytes $diskSize.Value -Path $newVhdPath -ErrorAction Stop
                     }
                     "Dynamic" {
                         $vhdFile = New-VHD -ComputerName $ComputerName -Dynamic -SizeBytes $diskSize.Value -Path $newVhdPath -ErrorAction Stop
                     }
                 }
                 $vhdFilePath = $vhdFile.Path
             }
             "Differencing" {
                 $newVhdPath = Join-Path -Path $defaultVirtualHardDiskPath -ChildPath "$Name.vhdx"
                 $vhdFile = New-VHD -ComputerName $ComputerName -Differencing -ParentPath $parentVhdPath.Value -Path $newVhdPath -ErrorAction Stop
                 $vhdFilePath = $vhdFile.Path
             }
         }
         if ($vhdFilePath -ne $null) {
             $command = "New-VM -ComputerName $ComputerName -Name '$Name' -SwitchName '$SwitchName' -VHDPath '$vhdFilePath'"
         } else {
             $command = "New-VM -ComputerName $ComputerName -Name '$Name' -SwitchName '$SwitchName' -NoVHD"
         }
         if ($Configuration -ne $null) {
             foreach ($configName in $Configuration.Keys.GetEnumerator()) {
                 if ($validConfigNames -contains $configName) {
                     $configParams += "-$configName" + " " + $Configuration[$configName]
                 }
             }
             $configParams = $configParams -join " "
         }
         if ($configParams.Count -eq 0) {
             $command += " -ErrorAction Stop"
         } else {
             $command += " $configParams -ErrorAction Stop"        
         }
         Try
         {
             Invoke-Expression -Command $command
         }
         Catch
         {
             $PSCmdlet.WriteError($Error[0])
             Remove-Item -Path $vhdFile.Path
         }
     }
     End {}
 }
`

