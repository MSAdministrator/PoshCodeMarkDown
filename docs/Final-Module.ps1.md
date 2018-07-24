---
Author: don jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5252
Published Date: 2014-06-20t14
Archived Date: 2014-06-25t06
---

# final - 

## Description

final script module

## Comments



## Usage



## TODO



## function

`get-osinfo`

## Code

`#
 #
 function Get-OSInfo {
 <#
 .SYNOPSIS
 Gets OS info.
 
 .DESCRIPTION
 Uses CIM, so PowerShell 3+ needs to be on all target nodes.
 
 .PARAMETER Computername
 The name of the computer. Duh.
 
 .PARAMETER ErrorLogFile
 Defaults to c:\errors.txt; this is the text file where
 the names of failed computers will be saved. This file
 will be deleted, if it exists, when the command runs.
 
 .EXAMPLE
 Get-OSInfo -Computername DC,S1
 This example gets Os info from two computers.
 
 .EXAMPLE
 "s1","s2" | Get-OSInfo
 This example gets OS info from two computers, but accepts
 their names from the pipeline.
 #>
     [CmdletBinding()]
     param(
 
         [Parameter(Mandatory=$True,
                    ValueFromPipeline=$True,
                    ValueFromPipelineByPropertyName=$True,
                    HelpMessage='Computer name to query')]
         [Alias('cn','hostname')]
         [string[]]$ComputerName,
 
         [Parameter(HelpMessage='Text file for failed computer names')]
         [string]$ErrorLogFile = 'c:\errors.txt'
     )
 
     BEGIN {
 
         Write-Verbose "Removing $ErrorLogFile"
         Remove-Item -Path $ErrorLogFile -ErrorAction SilentlyContinue
 
     }
 
     PROCESS {
 
         foreach ($computer in $computername) {
             try {
 
                 Write-Verbose "Connecting to $computer"
                 $os = Get-CimInstance -Class Win32_OperatingSystem -Comp $Computer  -ErrorAction Stop
                 $compsys =Get-CimInstance -Class Win32_ComputerSystem -Comp $Computer
 
                 switch ($compsys.Manufacturer) {
                     'Microsoft ' { $mfgr = 'MS' }
                     'Dell'       { $mfgr = 'Dell Computer' }
                     default      { $mfgr = $compsys.Manufacturer }
                 }
                 Write-Verbose "Manufacturer is now $mfgr, OS version is $($os.version)"
 
                 $output = @{'ComputerName' = $computer;
                             'OSVersion'    = $os.version;
                             'SPVersion'    = $os.ServicePackMajorVersion;
                             'Mfgr'         = $mfgr;
                             'Model'        = $compsys.Model}
 
                 $disks_array = @()
                 $disks = Get-CimInstance -ClassName Win32_LogicalDisk -ComputerName $Computer
                 foreach ($disk in $disks) {
                     $disk_object_props = @{'DriveLetter'=$disk.DeviceID;
                                            'Size'=$disk.Size;
                                            'FreeSpace'=$disk.FreeSpace}
                     $disk_object = New-Object -TypeName PSObject -Property $disk_object_props
                     $disks_array += $disk_object
                 }
 
                 $output.add('Disks',$disks_array)
 
                 $object = New-Object -TypeName PSObject -Property $output
                 $object.psobject.typenames.insert(0,'MyTools.OSInfo.Output.Thing')
                 Write-Output $object
 
             } catch {
 
                 $oops = $_
                 $Computer | out-file -FilePath $ErrorLogFile -Append
                 Write-Warning "OMG, $computer failed"
                 Write-Warning "Apparently, $oops"
 
 
 function Set-ServiceLogonPassword {
     [CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Medium')]
     param(
         [Parameter(Mandatory=$True,
                    ValueFromPipeline=$True,
                    ValueFromPipelineByPropertyName=$True)]
         [Alias('DNSHostName')]
         [string[]]$ComputerName,
 
         [Parameter(Mandatory=$True)]
         [string]$ServiceName,
 
         [Parameter(Mandatory=$True)]
         [string]$NewPassword
     )
     PROCESS {
         
         foreach ($computer in $computername) {
 
             $rv = Get-CimInstance  -ClassName Win32_Service `
                                    -Filter "name='$servicename'" `
                                    -ComputerName $computer |
                   Invoke-CimMethod -MethodName Change `
                                    -Arguments @{'StartPassword'=$NewPassword}
 
             if ($rv.returnvalue -ne 0) {
                 Write-warning "It didn't work on $computer for $servicename. Panic."
             }
 
         }
 
     }
 }
`

