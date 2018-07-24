---
Author: zachary loeber
Publisher: 
Copyright: 
Email: 
Version: 6.1.7601
Encoding: utf-8
License: cc0
PoshCode ID: 4367
Published Date: 2013-08-07t02
Archived Date: 2013-09-28t03
---

# get-computerassetinforma - 

## Description

a multi-threaded, windows asset gathering function.

## Comments



## Usage



## TODO



## function

`get-computerassetinformation`

## Code

`#
 #
 function Get-ComputerAssetInformation
 {
 <#
 .SYNOPSIS
    Get inventory data for specified computer systems.
 .DESCRIPTION
    Get inventory data for provided host using wmi. Data proccessing uses multithreading and supports using timeouts 
    in case of wmi problems. You can optionally include more detailed Drive, Memory, and Network information in the results.
    You can view verbose information on each runtime thread in realtime with the -Verbose option.
 .PARAMETER ComputerName
    Specifies the target computer for data query.
 .PARAMETER IncludeMemoryInfo
    Include information about the memory arrays and the installed memory within them. (_Memory and _MemoryArray)
 .PARAMETER IncludeDiskInfo
    Include disk partition and mount point information. (_Disks)
 .PARAMETER IncludeNetworkInfo
    Include general network configuration for enabled interfaces. (_Network)
 .PARAMETER ThrottleLimit
    Specifies the maximum number of systems to inventory simultaneously 
 .PARAMETER Timeout
    Specifies the maximum time in second command can run in background before terminating this thread.
 .PARAMETER ShowProgress
    Show progress bar information
 .PARAMETER PromptForCredential
    Prompt for remote system credential prior to processing request.
 .PARAMETER Credential
    Accept alternate credential (ignored if the localhost is processed)
 .EXAMPLE
    PS > Get-ComputerAssetInformation -ComputerName test1
  
         ComputerName        : TEST1
         Model               : ProLiant DL380 G7
         ChassisModel        : Rack Mount Unit
         OperatingSystem     : Microsoft Windows Server 2008 R2 Enterprise 
         OSServicePack       : 1
         OSVersion           : 6.1.7601
         OSSKU               : Enterprise Server Edition
         OSArchitecture      : x64
         SystemArchitecture  : x64
         PhysicalMemoryTotal : 12.0 GB
         PhysicalMemoryFree  : 621.7 MB
         VirtualMemoryTotal  : 24.0 GB
         VirtualMemoryFree   : 5.7 GB
         CPUCores            : 24
         CPUSockets          : 2
         SystemTime          : 08/04/2013 20:33:47
         LastBootTime        : 07/16/2013 07:42:01
         InstallDate         : 07/02/2011 17:52:34
         Uptime              : 19 days 12 hours 51 minutes
  
    Description
    -----------
    Query and display basic information ablout computer test1
 
 .EXAMPLE
    PS > $cred = Get-Credential
    PS > $b = Get-ComputerAssetInformation -ComputerName Test1 -Credential $cred -IncludeMemoryInfo 
    PS > $b | Select MemorySlotsTotal,MemorySlotsUsed | fl
 
    MemorySlotsTotal : 18
    MemorySlotsUsed  : 6
    
    PS > $b._Memory | Select DeviceLocator,@{n='MemorySize'; e={$_.Capacity/1Gb}}
 
    DeviceLocator                                                                   MemorySize
    -------------                                                                   ----------
    PROC 1 DIMM 3A                                                                           2
    PROC 1 DIMM 6B                                                                           2
    PROC 1 DIMM 9C                                                                           2
    PROC 2 DIMM 3A                                                                           2
    PROC 2 DIMM 6B                                                                           2
    PROC 2 DIMM 9C                                                                           2
    
    Description
    -----------
    Query information about computer test1 using alternate credentials, including detailed memory information. Return
    physical memory slots available and in use. Then display the memory location and size.
 
 .NOTES
    Originally posted at: http://learn-powershell.net/2013/05/08/scripting-games-2013-event-2-favorite-and-not-so-favorite/
    Extended and further hacked by: Zachary Loeber
    Site: http://www.the-little-things.net/
    Requires: Powershell 2.0
    Info: WMI prefered over CIM as there no speed advantage using cimsessions in multithreading against old systems. Starting
          around line 263 you can modify the WMI_<Property>Props arrays to include extra wmi data for each element should you
          require information I may have missed. You can also change the default display properties by modifying $defaultProperties.
          Keep in mind that including extra elements like the drive space and network information will increase the processing time per
          system. You may need to increase the timeout parameter accordingly.
    
    Version History
    1.1.0 - 8/3/2013
     - Added several parameters
     - Removed parameter sets in favor of arrays of custom object as note properties
     - Removed ICMP response requirements
     - Included more verbose runspace logging    
    1.0.2 - 8/2/2013
     - Split out system and OS architecture (changing how each it determined)
    1.0.1 - 8/1/2013
     - Updated to include several more bits of information and customization variables
    1.0.0 - ???
     - Discovered original script on the internet and totally was blown away at how awesome it is.
 #>
     [CmdletBinding()]
     Param
     (
         [Parameter(ValueFromPipeline=$true,
                    ValueFromPipelineByPropertyName=$true,
                    Position=0)]
         [ValidateNotNullOrEmpty()]
         [Alias('DNSHostName','PSComputerName')]
         [string[]]
         $ComputerName=$env:computername,
         
         [Parameter()]
         [switch]
         $IncludeMemoryInfo,
  
         [Parameter()]
         [switch]
         $IncludeDiskInfo,
  
         [Parameter()]
         [switch]
         $IncludeNetworkInfo,
        
         [Parameter()]
         [ValidateRange(1,65535)]
         [int32]
         $ThrottleLimit = 32,
  
         [Parameter()]
         [ValidateRange(1,65535)]
         [int32]
         $Timeout = 120,
  
         [Parameter()]
         [switch]
         $ShowProgress,
         
         [Parameter()]
         [switch]
         $PromptForCredential,
         
         [Parameter()]
         [System.Management.Automation.Credential()]
         $Credential = [System.Management.Automation.PSCredential]::Empty
     )
 
     Begin
     {
         Write-Verbose -Message 'Creating local hostname list'
         $IPAddresses = [net.dns]::GetHostAddresses($env:COMPUTERNAME) | Select-Object -ExpandProperty IpAddressToString
         $HostNames = $IPAddresses | ForEach-Object {
             try {
                 [net.dns]::GetHostByAddress($_)
             } catch {
             }
         } | Select-Object -ExpandProperty HostName -Unique
         $LocalHost = @('', '.', 'localhost', $env:COMPUTERNAME, '::1', '127.0.0.1') + $IPAddresses + $HostNames
  
         Write-Verbose -Message 'Creating initial variables'
         $runspacetimers       = [HashTable]::Synchronized(@{})
         $runspaces            = New-Object -TypeName System.Collections.ArrayList
         $bgRunspaceCounter    = 0
         
         if ($PromptForCredential)
         {
             $Credential = Get-Credential
         }
         
         Write-Verbose -Message 'Creating Initial Session State'
         $iss = [System.Management.Automation.Runspaces.InitialSessionState]::CreateDefault()
         foreach ($ExternalVariable in ('runspacetimers', 'Credential', 'LocalHost'))
         {
             Write-Verbose -Message "Adding variable $ExternalVariable to initial session state"
             $iss.Variables.Add((New-Object -TypeName System.Management.Automation.Runspaces.SessionStateVariableEntry -ArgumentList $ExternalVariable, (Get-Variable -Name $ExternalVariable -ValueOnly), ''))
         }
         
         Write-Verbose -Message 'Creating runspace pool'
         $rp = [System.Management.Automation.Runspaces.RunspaceFactory]::CreateRunspacePool(1, $ThrottleLimit, $iss, $Host)
         $rp.Open()
  
         Write-Verbose -Message 'Defining background runspaces scriptblock'
         $ScriptBlock = {
             [CmdletBinding()]
             Param
             (
                 [Parameter(Position=0)]
                 [string]
                 $ComputerName,
  
                 [Parameter(Position=1)]
                 [int]
                 $bgRunspaceID,
           
                 [Parameter()]
                 [switch]
                 $IncludeMemoryInfo,
          
                 [Parameter()]
                 [switch]
                 $IncludeDiskInfo,
          
                 [Parameter()]
                 [switch]
                 $IncludeNetworkInfo
             )
             $runspacetimers.$bgRunspaceID = Get-Date
             
             try
             {
                 Write-Verbose -Message ('Runspace {0}: Start' -f $ComputerName)
                 $WMIHast = @{
                     ComputerName = $ComputerName
                     ErrorAction = 'Stop'
                 }
                 if (($LocalHost -notcontains $ComputerName) -and ($Credential -ne $null))
                 {
                     $WMIHast.Credential = $Credential
                 }
 
                 $ReadableOutput = @{
                     Name = 'ToString'
                     MemberType = 'ScriptMethod'
                     PassThru = $true
                     Force = $true
                     Value = [ScriptBlock]::Create(@"
                         "{0:N1} {1}" -f @(
                             switch -Regex ([math]::Log(`$this,1024)) {
                                 ^0 {
                                     (`$this / 1), ' B'
                                 }
                                 ^1 {
                                     (`$this / 1KB), 'KB'
                                 }
                                 ^2 {
                                     (`$this / 1MB), 'MB'
                                 }
                                 ^3 {
                                     (`$this / 1GB), 'GB'
                                 }
                                 ^4 {
                                     (`$this / 1TB), 'TB'
                                 }
                                 default {
                                     (`$this / 1PB), 'PB'
                                 }
                             }
                         )
 "@
                     )
                 }
                 
                 Write-Verbose -Message ('Runspace {0}: General asset information' -f $ComputerName)
                 $SKUs                 = @("Undefined","Ultimate Edition","Home Basic Edition","Home Basic Premium Edition","Enterprise Edition",`
                                           "Home Basic N Edition","Business Edition","Standard Server Edition","DatacenterServer Edition","Small Business Server Edition",`
                                           "Enterprise Server Edition","Starter Edition","Datacenter Server Core Edition","Standard Server Core Edition",`
                                           "Enterprise ServerCoreEdition","Enterprise Server Edition for Itanium-Based Systems","Business N Edition","Web Server Edition",`
                                           "Cluster Server Edition","Home Server Edition","Storage Express Server Edition","Storage Standard Server Edition",`
                                           "Storage Workgroup Server Edition","Storage Enterprise Server Edition","Server For Small Business Edition","Small Business Server Premium Edition")
                 $ChassisModels        = @("PlaceHolder","Maybe Virtual Machine","Unknown","Desktop","Thin Desktop","Pizza Box","Mini Tower","Full Tower","Portable",`
                                           "Laptop","Notebook","Hand Held","Docking Station","All in One","Sub Notebook","Space-Saving","Lunch Box","Main System Chassis",`
                                           "Lunch Box","SubChassis","Bus Expansion Chassis","Peripheral Chassis","Storage Chassis" ,"Rack Mount Unit","Sealed-Case PC")
                 $defaultProperties    = @('ComputerName','Model','ChassisModel','OperatingSystem','OSServicePack','OSVersion','OSSKU','OSArchitecture', `
                                           'SystemArchitecture','PhysicalMemoryTotal','PhysicalMemoryFree','VirtualMemoryTotal','VirtualMemoryFree', `
                                           'CPUCores','CPUSockets','SystemTime','LastBootTime','InstallDate','Uptime')
                 $WMI_OSProps          = @('BuildNumber','Version','SerialNumber','ServicePackMajorVersion','CSDVersion','SystemDrive',`
                                           'SystemDirectory','WindowsDirectory','Caption','TotalVisibleMemorySize','FreePhysicalMemory',`
                                           'TotalVirtualMemorySize','FreeVirtualMemory','OSArchitecture','Organization','LocalDateTime',`
                                           'RegisteredUser','OperatingSystemSKU','OSType','LastBootUpTime','InstallDate')
                 $WMI_ProcProps        = @('Name','Description','MaxClockSpeed','CurrentClockSpeed','AddressWidth','NumberOfCores','NumberOfLogicalProcessors', `
                                           'DataWidth')
                 $WMI_CompProps        = @('DNSHostName','Domain','Manufacturer','Model','NumberOfLogicalProcessors','NumberOfProcessors','PrimaryOwnerContact', `
                                           'PrimaryOwnerName','TotalPhysicalMemory')
                 $WMI_ChassisProps     = @('ChassisTypes','Manufacturer','SerialNumber','Tag','SKU')
                 $wmi_compsystem = Get-WmiObject @WMIHast -Class Win32_ComputerSystem | select $WMI_CompProps
                 $wmi_os = Get-WmiObject @WMIHast -Class Win32_OperatingSystem | select $WMI_OSProps
                 $wmi_proc = Get-WmiObject @WMIHast -Class Win32_Processor | select $WMI_ProcProps
                 $wmi_chassis = Get-WmiObject @WMIHast -Class Win32_SystemEnclosure | select $WMI_ChassisProps
 
                 {
                     $Sockets = @($wmi_proc).Count
                     $Cores = ($wmi_proc | Measure-Object -Property NumberOfLogicalProcessors -Sum).Sum
                 }
                 {
                     $Sockets = @($wmi_proc | Select-Object -Property SocketDesignation -Unique).Count
                     $Cores = @($wmi_proc).Count
                 }
                 
                 if ($wmi_os.OperatingSystemSKU -ne $null)
                 {
                     $OS_SKU = $SKUs[$wmi_os.OperatingSystemSKU]
                 }
                 else
                 {
                     $OS_SKU = 'Not Available'
                 }
                 $System_Time = ([wmi]'').ConvertToDateTime($wmi_os.LocalDateTime).tostring("MM/dd/yyyy HH:mm:ss")
                 $OS_LastBoot = ([wmi]'').ConvertToDateTime($wmi_os.LastBootUptime).tostring("MM/dd/yyyy HH:mm:ss")
                 $OS_InstallDate = ([wmi]'').ConvertToDateTime($wmi_os.InstallDate).tostring("MM/dd/yyyy HH:mm:ss")
                 $Uptime = New-TimeSpan -Start $OS_LastBoot -End $System_Time                    
 
                 $ResultProperty = @{
                     'PSComputerName' = $ComputerName
                     'Model' = $wmi_compsystem.Model
                     'ChassisModel' = $ChassisModels[$wmi_chassis.ChassisTypes[0]]
                     'ComputerName' = $wmi_compsystem.DNSHostName                        
                     'OperatingSystem' = $wmi_os.Caption
                     'OSServicePack' = $wmi_os.ServicePackMajorVersion
                     'OSVersion' = $wmi_os.Version
                     'OSSKU' = $OS_SKU
                     'OSArchitecture' = "x$($wmi_proc.AddressWidth[0])"
                     'SystemArchitecture' = "x$($wmi_proc.DataWidth[0])"
                     'PhysicalMemoryTotal' = ($wmi_os.TotalVisibleMemorySize * 1024) | Add-Member @ReadableOutput
                     'PhysicalMemoryFree' = ($wmi_os.FreePhysicalMemory * 1024) | Add-Member @ReadableOutput
                     'VirtualMemoryTotal' = ($wmi_os.TotalVirtualMemorySize * 1024) | Add-Member @ReadableOutput
                     'VirtualMemoryFree' = ($wmi_os.FreeVirtualMemory * 1024) | Add-Member @ReadableOutput
                     'CPUCores' = $Cores
                     'CPUSockets' = $Sockets
                     'LastBootTime' = $OS_LastBoot
                     'InstallDate' = $OS_InstallDate
                     'SystemTime' = $System_Time
                     'Uptime' = "$($Uptime.days) days $($Uptime.hours) hours $($Uptime.minutes) minutes"
                     '_OS' = $wmi_os
                     '_System' = $wmi_compsystem
                     '_Processor' = $wmi_proc
                     '_Chassis' = $wmi_chassis
                 }
                 
                 if ($IncludeMemoryInfo)
                 {
                     Write-Verbose -Message ('Runspace {0}: Memory information' -f $ComputerName)
                     $WMI_MemProps = @('BankLabel','DeviceLocator','Capacity','PartNumber','Speed','Tag')
                     $WMI_MemArrayProps = @('Tag','MemoryDevices','MaxCapacity')
                     $wmi_memory = Get-WmiObject @WMIHast -Class Win32_PhysicalMemory | select $WMI_MemProps
                     $wmi_memoryarray = Get-WmiObject @WMIHast -Class Win32_PhysicalMemoryArray | select $WMI_MemArrayProps
                     
                     $Memory_Slotstotal = 0
                     $Memory_SlotsUsed = (@($wmi_memory)).Count                
                     @($wmi_memoryarray) | % {$Memory_Slotstotal = $Memory_Slotstotal + $_.MemoryDevices}
                     
                     $ResultProperty.MemorySlotsTotal = $Memory_Slotstotal
                     $ResultProperty.MemorySlotsUsed = $Memory_SlotsUsed
                     $ResultProperty._MemoryArray = $wmi_memoryarray
                     $ResultProperty._Memory = $wmi_memory
                 }
                 
                 if ($IncludeNetworkInfo)
                 {
                     Write-Verbose -Message ('Runspace {0}: Network information' -f $ComputerName)
                     $WMI_NetProps = @('Index','IpAddress','IpSubnet','MACAddress','DefaultIPGateway','Description', `
                                       'IPEnabled','InterfaceIndex','DHCPEnabled','DHCPServer', `
                                       'DNSDomain','DNSDomainSuffixSearchOrder','DomainDNSRegistrationEnabled ', `
                                       'WinsPrimaryServer', 'WinsSecondaryServer')
                     $wmi_net = Get-WmiObject @WMIHast -Class Win32_NetworkAdapterConfiguration -Filter "IPEnabled='True'" | `
                                select $WMI_NetProps
                     $ResultProperty._Network = $wmi_net
                 }                    
                 
                 if ($IncludeDiskInfo)
                 {
                     Write-Verbose -Message ('Runspace {0}: Disk information' -f $ComputerName)
                     $WMI_DiskPartProps    = @('DiskIndex','Index','Name','DriveLetter','Caption','Capacity','FreeSpace','SerialNumber')
                     $WMI_DiskVolProps     = @('Name','DriveLetter','Caption','Capacity','FreeSpace','SerialNumber')
                     $WMI_DiskMountProps   = @('Name','Label','Caption','Capacity','FreeSpace','Compressed','PageFilePresent','SerialNumber')
                     
                     $wmi_diskdrives = Get-WmiObject @WMIHast -Class Win32_DiskDrive | select $WMI_DiskDriveProps
                     #$wmi_diskvolumes = Get-WmiObject @WMIHast -Class Win32_Volume -Filter "DriveLetter IS NOT NULL" | select $WMI_DiskVolProps
                     $wmi_mountpoints = Get-WmiObject @WMIHast -Class Win32_Volume -Filter "DriveType=3 AND DriveLetter IS NULL" | select $WMI_DiskMountProps
                     
                     $AllDisks = @()
                     foreach ($diskdrive in $wmi_diskdrives) 
                     {
                         $partitionquery = "ASSOCIATORS OF {Win32_DiskDrive.DeviceID=`"$($diskdrive.DeviceID.replace('\','\\'))`"} WHERE AssocClass = Win32_DiskDriveToDiskPartition"
                         $partitions = @(Get-WmiObject @WMIHast -Query $partitionquery)
                         foreach ($partition in $partitions)
                         {
                             $logicaldiskquery = "ASSOCIATORS OF {Win32_DiskPartition.DeviceID=`"$($partition.DeviceID)`"} WHERE AssocClass = Win32_LogicalDiskToPartition"
                             $logicaldisks = @(Get-WmiObject @WMIHast -Query $logicaldiskquery)
                             foreach ($logicaldisk in $logicaldisks)
                             {
                                $diskprops = @{
                                                Disk = $diskdrive.Name
                                                Model = $diskdrive.Model
                                                Partition = $partition.Name
                                                Description = $partition.Description
                                                PrimaryPartition = $partition.PrimaryPartition
                                                VolumeName = $logicaldisk.VolumeName
                                                Drive = $logicaldisk.Name
                                                DiskSize = $logicaldisk.Size | Add-Member @ReadableOutput
                                                FreeSpace = $logicaldisk.FreeSpace | Add-Member @ReadableOutput
                                                PercentageFree = [math]::round((($logicaldisk.FreeSpace/$logicaldisk.Size)*100), 2)
                                                DiskType = 'Partition'
                                                SerialNumber = $diskdrive.SerialNumber
                                              }
                                 $AllDisks += New-Object psobject -Property $diskprops
                             }
                         }
                     }
                     foreach ($mountpoint in $wmi_mountpoints)
                     {                    
                         $diskprops = @{
                                Disk = $mountpoint.Name
                                Model = ''
                                Partition = ''
                                Description = $mountpoint.Caption
                                PrimaryPartition = ''
                                VolumeName = ''
                                VolumeSerialNumber = ''
                                Drive = [Regex]::Match($mountpoint.Caption, "^.:\\").Value
                                DiskSize = $mountpoint.Capacity  | Add-Member @ReadableOutput
                                FreeSpace = $mountpoint.FreeSpace  | Add-Member @ReadableOutput
                                PercentageFree = [math]::round((($mountpoint.FreeSpace/$mountpoint.Capacity)*100), 2)
                                DiskType = 'MountPoint'
                                SerialNumber = $mountpoint.SerialNumber
                              }
                         $AllDisks += New-Object psobject -Property $diskprops
                     }
                     $ResultProperty._Disks = $AllDisks
                 }
                 
                 $ResultObject = New-Object -TypeName PSObject -Property $ResultProperty
 
                 $ResultObject.PSObject.TypeNames.Insert(0,'My.Asset.Info')
                 $defaultDisplayPropertySet = New-Object System.Management.Automation.PSPropertySet(�DefaultDisplayPropertySet�,[string[]]$defaultProperties)
                 $PSStandardMembers = [System.Management.Automation.PSMemberInfo[]]@($defaultDisplayPropertySet)
                 $ResultObject | Add-Member MemberSet PSStandardMembers $PSStandardMembers
 
                 Write-Output -InputObject $ResultObject
             }
             catch
             {
                 Write-Warning -Message ('{0}: {1}' -f $ComputerName, $_.Exception.Message)
             }
             Write-Verbose -Message ('Runspace {0}: End' -f $ComputerName)
         }
  
         function Get-Result
         {
             [CmdletBinding()]
             Param 
             (
                 [switch]$Wait
             )
             do
             {
                 $More = $false
                 foreach ($runspace in $runspaces)
                 {
                     $StartTime = $runspacetimers.($runspace.ID)
                     if ($runspace.Handle.isCompleted)
                     {
                         Write-Verbose -Message ('Thread done for {0}' -f $runspace.IObject)
                         $runspace.PowerShell.EndInvoke($runspace.Handle)
                         $runspace.PowerShell.Dispose()
                         $runspace.PowerShell = $null
                         $runspace.Handle = $null
                     }
                     elseif ($runspace.Handle -ne $null)
                     {
                         $More = $true
                     }
                     if ($Timeout -and $StartTime)
                     {
                         if ((New-TimeSpan -Start $StartTime).TotalSeconds -ge $Timeout -and $runspace.PowerShell)
                         {
                             Write-Warning -Message ('Timeout {0}' -f $runspace.IObject)
                             $runspace.PowerShell.Dispose()
                             $runspace.PowerShell = $null
                             $runspace.Handle = $null
                         }
                     }
                 }
                 if ($More -and $PSBoundParameters['Wait'])
                 {
                     Start-Sleep -Milliseconds 100
                 }
                 foreach ($threat in $runspaces.Clone())
                 {
                     if ( -not $threat.handle)
                     {
                         Write-Verbose -Message ('Removing {0} from runspaces' -f $threat.IObject)
                         $runspaces.Remove($threat)
                     }
                 }
                 if ($ShowProgress)
                 {
                     $ProgressSplatting = @{
                         Activity = 'Getting asset info'
                         Status = '{0} of {1} total threads done' -f ($bgRunspaceCounter - $runspaces.Count), $bgRunspaceCounter
                         PercentComplete = ($bgRunspaceCounter - $runspaces.Count) / $bgRunspaceCounter * 100
                     }
                     Write-Progress @ProgressSplatting
                 }
             }
             while ($More -and $PSBoundParameters['Wait'])
         }
     }
     Process
     {
         foreach ($Computer in $ComputerName)
         {
             $bgRunspaceCounter++
             #$psCMD = [System.Management.Automation.PowerShell]::Create().AddScript($ScriptBlock).AddParameter('bgRunspaceID',$bgRunspaceCounter).AddParameter('ComputerName',$Computer).AddParameter('IncludeMemoryInfo',$IncludeMemoryInfo).AddParameter('IncludeDiskInfo',$IncludeDiskInfo).AddParameter('IncludeNetworkInfo',$IncludeNetworkInfo).AddParameter('Verbose',$Verbose)
             $psCMD = [System.Management.Automation.PowerShell]::Create().AddScript($ScriptBlock)
             $null = $psCMD.AddParameter('bgRunspaceID',$bgRunspaceCounter)
             $null = $psCMD.AddParameter('ComputerName',$Computer)
             $null = $psCMD.AddParameter('IncludeMemoryInfo',$IncludeMemoryInfo)
             $null = $psCMD.AddParameter('IncludeDiskInfo',$IncludeDiskInfo)
             $null = $psCMD.AddParameter('IncludeNetworkInfo',$IncludeNetworkInfo)
             $psCMD.RunspacePool = $rp
  
             Write-Verbose -Message ('Starting {0}' -f $Computer)
             [void]$runspaces.Add(@{
                 Handle = $psCMD.BeginInvoke()
                 PowerShell = $psCMD
                 IObject = $Computer
                 ID = $bgRunspaceCounter
                 })
            Get-Result
         }
     }
  
     End
     {
         Get-Result -Wait
         if ($ShowProgress)
         {
             Write-Progress -Activity 'Getting asset info' -Status 'Done' -Completed
         }
         Write-Verbose -Message "Closing runspace pool"
         $rp.Close()
         $rp.Dispose()
     }
 }
`

