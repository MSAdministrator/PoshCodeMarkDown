---
Author: zachary loeber
Publisher: 
Copyright: 
Email: 
Version: 6.2.9200
Encoding: utf-8
License: cc0
PoshCode ID: 4355
Published Date: 2015-08-02t02
Archived Date: 2015-05-04t22
---

# get-assetinfo - 

## Description

a multi-threaded, windows asset gathering function.

## Comments



## Usage



## TODO



## script

`get-assetinfo`

## Code

`#
 #
 <#
 .SYNOPSIS
    Get inventory data for specified computer system.
 .DESCRIPTION
    Get inventory data for provided host using wmi.
    Data proccessing use multithreading and support using timeouts in case of wmi problems.
    Target computer system must be reacheble using ICMP Echo.
    Provide ComputerName specified by user and HostName used by OS. Also provide OS version, CPU and memory info.
 .PARAMETER ComputerName
    Specifies the target computer for data query.
 .PARAMETER ThrottleLimit
    Specifies the maximum number of systems to inventory simultaneously 
 .PARAMETER Timeout
    Specifies the maximum time in second command can run in background before terminating this thread.
 .PARAMETER ShowProgress
    Show progress bar information
 .EXAMPLE
    PS > Get-AssetInfo -ComputerName test1
  
    ComputerName : hp-test1
    OSCaption    : Microsoft Windows 8 Enterprise
    Memory       : 5,93 GB
    Cores        : 2
    Sockets      : 1
  
    Description
    -----------
    Query information ablout computer test1
 .EXAMPLE
    PS > Get-AssetInfo -ComputerName test1 -Credential (get-credential) | fromat-list * -force
  
    ComputerName   : hp-test1
    OSCaption      : Microsoft Windows 8 Enterprise
    OSVersion      : 6.2.9200
    Cores          : 2
    OSServicePack  : 0
    Memory         : 5,93 GB
    Sockets        : 1
    PSComputerName : test1
    Description
    -----------
    Query information ablout computer test1 using alternate credentials
 .EXAMPLE
    PS > get-content C:\complist.txt | Get-AssetInfo -ThrottleLimit 100 -Timeout 60 -ShowProgress
  
    Description
    -----------
    Query information about computers in file C:\complist.txt using 100 thread at time with 60 sec timeout and showing progressbar
 .EXAMPLE
    PS > $a = Get-AssetInfo
    PS > $a | Select Memory,Chassis
    
    Description
    -----------
    Query information about the  local computer, store in $a and then show output for the Memory and Chassis property groups
 
 .NOTES
    Originally posted at: http://learn-powershell.net/2013/05/08/scripting-games-2013-event-2-favorite-and-not-so-favorite/
    Extended and further hacked by: Zachary Loeber
    Site: http://www.the-little-things.net/
    Requires: Powershell 2.0
    Info: WMI prefered over CIM as there no speed advantage using cimsessions in multitheating against old systems.
    The following are the default property groups you can send to output:
     - Default
     - System
     - OS
     - Processor
     - Chassis
     - Memory
     - MemoryArray
     - Network
 #>
 function Get-AssetInfo
 {
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
  
         [Parameter(Position=1)]
         [ValidateRange(1,65535)]
         [int32]
         $ThrottleLimit = 32,
  
         [Parameter(Position=2)]
         [ValidateRange(1,65535)]
         [int32]
         $Timeout = 120,
  
         [Parameter(Position=3)]
         [switch]
         $ShowProgress,
  
         [Parameter(Position=4)]
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
                 $bgRunspaceID
             )
             $runspacetimers.$bgRunspaceID = Get-Date
             
             function Join-Object ($inputobj, $objtoadd, $prefix) {
                 $result = $inputobj
                 foreach($prop in $objtoadd | get-member -type Properties | select -expand Name) {
                     Add-Member -in $result -type NoteProperty -name "$($prefix)$($prop)" -value $objtoadd.($prop)
                 }
                 $result
             }
 
             function Prefix-StringArray ($array, $prefix) {
                 $result = [String[]]@()
                 foreach($item in $array) {
                     $result = $result + "$($prefix)$($item)"
                 }
                 $result
             }
             
             if (Test-Connection -ComputerName $ComputerName -Quiet -Count 1 -ErrorAction SilentlyContinue)
             {
                 try
                 {
                     Write-Verbose -Message "WMI Query: $ComputerName"
                     $WMIHast = @{
                         ComputerName = $ComputerName
                         ErrorAction = 'Stop'
                     }
                     if ($LocalHost -notcontains $ComputerName)
                     {
                         $WMIHast.Credential = $Credential
                     }
                     $WMI_OSProps          = @('BuildNumber','Version','SerialNumber','ServicePackMajorVersion','CSDVersion','SystemDrive',`
                                               'SystemDirectory','WindowsDirectory','Caption','TotalVisibleMemorySize','FreePhysicalMemory',`
                                               'TotalVirtualMemorySize','FreeVirtualMemory','OSArchitecture','Organization','LocalDateTime',`
                                               'RegisteredUser','OperatingSystemSKU','OSType','LastBootUpTime','InstallDate')
                     $prefix_OSProps       = 'OS_'
                     $WMI_ProcProps        = @('Name','Description','MaxClockSpeed','CurrentClockSpeed','AddressWidth','NumberOfCores','NumberOfLogicalProcessors')
                     $prefix_ProcProps     = 'CPU_'
                     $WMI_CompProps        = @('DNSHostName','Domain','Manufacturer','Model','NumberOfLogicalProcessors','NumberOfProcessors','PrimaryOwnerContact', `
                                               'PrimaryOwnerName','SystemType','TotalPhysicalMemory')
                     $prefix_CompProps     = 'System_'
                     $WMI_ChassisProps     = @('ChassisTypes','Manufacturer','SerialNumber','Tag','SKU')
                     $prefix_ChassisProps  =   'Chassis_'
                     $WMI_MemProps         = @('BankLabel','DeviceLocator','Capacity','PartNumber','Speed','Tag')
                     $prefix_MemProps      = 'Memory_'
                     $WMI_MemArrayProps    = @('Tag','MemoryDevices','MaxCapacity')
                     $prefix_MemArrayProps = 'MemoryArray_'
                     $WMI_NetProps         = @('Description', 'DHCPServer','IpAddress','IpSubnet','DefaultIPGateway','DNSServerSearchOrder','WinsPrimaryServer', `
                                               'WinsSecondaryServer')
                     $prefix_NetProps      = 'Net_'
                     $defaultProperties    = @('ComputerName','OSCaption','OSServicePack','OSVersion','OSSKU','Architecture', `
                                               'PhysicalMemoryTotal','PhysicalMemoryFree','VirtualMemoryTotal','VirtualMemoryFree', `
                                               'CPUCores','CPUSockets','MemorySlotsTotal','MemorySlotsUsed','SystemTime', `
                                               'LastBootTime','InstallDate','Uptime')
                     $SKUs                 = @("Undefined","Ultimate Edition","Home Basic Edition","Home Basic Premium Edition","Enterprise Edition",`
                                               "Home Basic N Edition","Business Edition","Standard Server Edition","DatacenterServer Edition","Small Business Server Edition",`
                                               "Enterprise Server Edition","Starter Edition","Datacenter Server Core Edition","Standard Server Core Edition",`
                                               "Enterprise ServerCoreEdition","Enterprise Server Edition for Itanium-Based Systems","Business N Edition","Web Server Edition",`
                                               "Cluster Server Edition","Home Server Edition","Storage Express Server Edition","Storage Standard Server Edition",`
                                               "Storage Workgroup Server Edition","Storage Enterprise Server Edition","Server For Small Business Edition","Small Business Server Premium Edition")
                     $ChassisModels        = @("PlaceHolder","Maybe Virtual Machine","Unknown","Desktop","Thin Desktop","Pizza Box","Mini Tower","Full Tower","Portable",`
                                               "Laptop","Notebook","Hand Held","Docking Station","All in One","Sub Notebook","Space-Saving","Lunch Box","Main System Chassis",`
                                               "Lunch Box","SubChassis","Bus Expansion Chassis","Peripheral Chassis","Storage Chassis" ,"Rack Mount Unit","Sealed-Case PC")
                     
                     $wmi_compsystem = Get-WmiObject @WMIHast -Class Win32_ComputerSystem | select $WMI_CompProps
                     $wmi_os = Get-WmiObject @WMIHast -Class Win32_OperatingSystem | select $WMI_OSProps
                     $wmi_proc = Get-WmiObject @WMIHast -Class Win32_Processor | select $WMI_ProcProps
                     $wmi_chassis = Get-WmiObject @WMIHast -Class Win32_SystemEnclosure | select $WMI_ChassisProps
                     $wmi_memory = Get-WmiObject @WMIHast -Class Win32_PhysicalMemory | select $WMI_MemProps
                     $wmi_memoryarray = Get-WmiObject @WMIHast -Class Win32_PhysicalMemoryArray | select $WMI_MemArrayProps
                     $wmi_net = Get-WmiObject @WMIHast -Class Win32_NetworkAdapterConfiguration | select $WMI_NetProps
                     
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
                     $Memory_Slotstotal = 0
                     $Memory_SlotsUsed = (@($wmi_memory)).Count                
                     @($wmi_memoryarray) | % {$Memory_Slotstotal = $Memory_Slotstotal + $_.MemoryDevices}
                     
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
 
                     $myObject = New-Object -TypeName PSObject -Property @{
                         'PSComputerName' = $ComputerName
                         'ComputerName' = $wmi_compsystem.DNSHostName                        
                         'OSCaption' = $wmi_os.Caption
                         'OSServicePack' = $wmi_os.ServicePackMajorVersion
                         'OSVersion' = $wmi_os.Version
                         'OSSKU' = $OS_SKU
                         'Architecture' = $wmi_os.OSArchitecture
                         'PhysicalMemoryTotal' = $wmi_os.TotalVisibleMemorySize | Add-Member @ReadableOutput
                         'PhysicalMemoryFree' = $wmi_os.FreePhysicalMemory | Add-Member @ReadableOutput
                         'VirtualMemoryTotal' = $wmi_os.TotalVirtualMemorySize | Add-Member @ReadableOutput
                         'VirtualMemoryFree' = $wmi_os.FreeVirtualMemory | Add-Member @ReadableOutput
                         'CPUCores' = $Cores
                         'CPUSockets' = $Sockets
                         'MemorySlotsTotal' = $Memory_Slotstotal
                         'MemorySlotsUsed' = $Memory_SlotsUsed
                         'SystemTime' = $System_Time
                         'LastBootTime' = $OS_LastBoot
                         'InstallDate' = $OS_InstallDate
                         'Uptime' = "$($Uptime.days) days $($Uptime.hours) hours $($Uptime.minutes) minutes"
                     }
 
                     $myObject = Join-Object $myObject $wmi_compsystem $prefix_CompProps
                     $myObject = Join-Object $myObject $wmi_os $prefix_OSProps
                     $myObject = Join-Object $myObject $wmi_proc $prefix_ProcProps
                     $myObject = Join-Object $myObject $wmi_chassis $prefix_ChassisProps
                     $myObject = Join-Object $myObject $wmi_memory $prefix_MemProps
                     $myObject = Join-Object $myObject $wmi_memoryarray $prefix_MemArrayProps
                     $myObject = Join-Object $myObject $wmi_net $prefix_NetProps
 
                     $myObject.PSObject.TypeNames.Insert(0,'My.Asset.Info')
                     $defaultDisplayPropertySet = New-Object System.Management.Automation.PSPropertySet(�DefaultDisplayPropertySet�,[string[]]$defaultProperties)
                     $PSStandardMembers = [System.Management.Automation.PSMemberInfo[]]@($defaultDisplayPropertySet)
                     $myObject | Add-Member MemberSet PSStandardMembers $PSStandardMembers
                     
                     $myObject | Add-Member PropertySet "Default" ([string[]]@($defaultProperties))
                     $myObject | Add-Member PropertySet "OS" ([string[]]@(Prefix-StringArray $WMI_OSProps $prefix_OSProps))
                     $myObject | Add-Member PropertySet "System" ([string[]]@(Prefix-StringArray $WMI_CompProps $prefix_CompProps))
                     $myObject | Add-Member PropertySet "Processor" ([string[]]@(Prefix-StringArray $WMI_ProcProps $prefix_ProcProps))
                     $myObject | Add-Member PropertySet "Chassis" ([string[]]@(Prefix-StringArray $WMI_ChassisProps $prefix_ChassisProps))
                     $myObject | Add-Member PropertySet "Memory" ([string[]]@(Prefix-StringArray $WMI_MemProps $prefix_MemProps))
                     $myObject | Add-Member PropertySet "MemoryArray" ([string[]]@(Prefix-StringArray $WMI_MemArrayProps $prefix_MemArrayProps))
                     $myObject | Add-Member PropertySet "Network" ([string[]]@(Prefix-StringArray $WMI_NetProps $prefix_NetProps))
 
                     Write-Output -InputObject $myObject
                 }
                 catch
                 {
                     Write-Warning -Message ('{0}: {1}' -f $ComputerName, $_.Exception.Message)
                 }
             }
             else
             {
                 Write-Warning -Message ("{0}: Unavailable" -f $ComputerName)
             }
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
             $psCMD = [System.Management.Automation.PowerShell]::Create().AddScript($ScriptBlock).AddParameter('bgRunspaceID',$bgRunspaceCounter).AddParameter('ComputerName',$Computer)
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

