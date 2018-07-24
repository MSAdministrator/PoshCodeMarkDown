---
Author: don jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5455
Published Date: 2014-09-19t09
Archived Date: 2014-09-25t15
---

# final elsa module - 

## Description

last module from class. see, i didn’t forget.

## Comments



## Usage



## TODO



## function

`get-donsysinfo`

## Code

`#
 #
 
 function Get-DonSysInfo {
 <#
 .SYNOPSIS
 Gets computer info. From computers. Many, many computers.
 
 .DESCRIPTION
 This uses CIM, so you can only query machines that have
 PowerShell v3 or later. Apart from that, it just works.
 
 .PARAMETER ComputerName
 The name of the computer(s). Accepts pipeline input.
 
 .PARAMETER ErrorLogFileName
 The file to write failed computer names to.
 
 .PARAMETER LeaveSession
 Use this to leave the CIM sessions open.
 
 .EXAMPLE
 Get-DonSysInfo -ComputerName ONE,TWO
 This example queries information from two computers.
 
 .EXAMPLE
 "DON","JOE" | Get-DonSysInfo
 This example uses pipeline input, ByValue, to query two computers.
 #>
     [CmdletBinding()]
     param(
         [Parameter(Mandatory=$True,
                    ValueFromPipeline=$True,
                    ValueFromPipelineByPropertyName=$True)]
         [Alias('DNSHostName')]
         [ValidateLength(1,10)]
         [string[]]$ComputerName,
 
         [Parameter()]
         [ValidateScript({-not (Test-Path $_)})]
         [string]$ErrorLogFileName = 'c:\failed-computers.txt',
 
         [switch]$LeaveSession
     )
     BEGIN {
         Write-Verbose "Erasing $ErrorLogFileName"
         Remove-Item -Path $ErrorLogFileName -ErrorAction SilentlyContinue
     
     }
     PROCESS {
 
         foreach ($computer in $computername) {
             try {
                 Write-Verbose "Connecting to $computer"
                 $session = New-CimSession -ComputerName $Computer -EA Stop
 
                 $proc = Get-CimInstance -CimSession $session -ClassName Win32_Processor 
                 $os = Get-CimInstance -CimSession $session -ClassName Win32_OperatingSystem
                 $cs = Get-CimInstance -CimSession $session -ClassName Win32_ComputerSystem
                 $disk = Get-CimInstance -CimSession $session -ClassName Win32_LogicalDisk -Filter "DeviceID='c:'"
 
                 Write-Verbose "Query complete, building output object for $computer"
                 $properties = @{'ComputerName'=$computer;
                                 'TotalCores'=($proc | Measure -Property NumberOfCores -sum | select -ExpandProperty sum);
                                 'RAM'=$cs.TotalPhysicalMemory;
                                 'OSArchitecture'=$os.OSArchitecture;
                                 'FreeOnC'=$disk.FreeSpace}
                 $obj = New-Object -TypeName PSObject -Property $properties
                 $obj.psobject.TypeNames.Insert(0,'Don.Awesome.SysInfo')
                 Write-Output $obj
 
                 if (-not ($LeaveSession)) { 
                     Write-Verbose "Closing session for $computer"
                     $session | Remove-CimSession
                 }
             } catch {
                 $problem = $_
                 Write-Warning "OMG, $computer FAIL FAIL FAIL"
                 Write-Warning "The problem was $problem"
                 $computer | Out-File -FilePath $ErrorLogFileName -Append
 
     END {}
 
 function Get-ServiceProcess {
     [CmdletBinding()]
     param(
         [string[]]$ComputerName
     )
 
     foreach ($computer in $ComputerName) {
 
         $services = Get-CimInstance -ClassName Win32_Service -ComputerName $computer -Filter "state='running'"
         Write-Debug "Finished querying services from $computer"
 
         foreach ($service in $services) {
 
             $process = Get-CimInstance -ClassName Win32_process `
                                        -ComputerName $computer `
                                        -Filter "processid=$($service.processid)"
 
             $properties = @{'ComputerName'=$Computer;
                             'ProcessID'=$process.ProcessId;
                             'ServiceName'=$service.Name}
             $obj = New-Object -TypeName PSObject -Property $properties
             Write-Debug "Ready to output object for $computer"
 
             Write-Output $obj
 
     
 function Set-ServicePassword {
     [CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Medium')]
     param(
         
         [Parameter(Mandatory=$True)]
         [string[]]$ComputerName,
 
         [Parameter(Mandatory=$True)]
         [string]$NewPassword,
 
         [Parameter(Mandatory=$True)]
         [string]$ServiceName
 
     )
     foreach ($computer in $computername) {
         $services = Get-WmiObject -Class Win32_Service -Filter "name='$servicename'" -ComputerName $computer
         foreach ($service in $services) {
 
             if ($PSCmdlet.ShouldProcess("Service $servicename on $computer")) {
                 $service.change($null,$null,$null,$null,$null,$null,$null,$newpassword)
              
 
 
 function Set-ComputerState {
     [CmdletBinding(SupportsShouldProcess=$True,ConfirmImpact='Medium')]
     param(
         [Parameter(Mandatory=$True,ValueFromPipeline=$True)]
         [string[]]$ComputerName,
         [switch]$Force,
         [Parameter(Mandatory=$True)]
         [ValidateSet('LogOff','Shutdown','PowerOff','Restart')]
         [string]$Action
     )
     BEGIN {
         switch ($Action) {
             'PowerOff' { $_action = 8 }
             'Shutdown' { $_action = 1 }
             'LogOff'   { $_action = 0 }
             'Restart'  { $_action = 2 }
         }
         if ($force) { $_action += 4 }
     }
     PROCESS {
         foreach ($computer in $ComputerName) {
             try {
                 $os = Get-WmiObject -Class Win32_OperatingSystem -ComputerName $computer -ErrorAction Stop
                 if ($PSCmdlet.ShouldProcess("$action on $computer")) {
                     $ret = $os.Win32Shutdown($_action)
                     if ($ret.returnvalue -ne 0) {
                         Write-Warning "Didn't work on $computer"
                     }
                 }
             } catch {
                 Write-Warning "Didn't connect to $computer"
             }
         }
     }
 
 function Get-SystemArchitecture {
     [CmdletBinding()]
     Param(
         [Parameter(Mandatory=$True,ValueFromPipeline=$True)]
         [string[]]$ComputerName
     )
     PROCESS {
         foreach ($computer in $ComputerName) {
             Write-Verbose "Querying Win32_OperatingSystem from $computer"
             $os = Get-WmiObject -Class Win32_OperatingSystem -ComputerName $computer
 
             Write-Verbose "Querying first instance of Win32_Processor from $computer"
             $proc = Get-WmiObject -Class Win32_Processor -ComputerName $computer |
                     Select -first 1
 
             Write-Verbose "Creating output object"
             $properties = @{
                 'ComputerName' = $computer;
                 'OSArchitecture' = $os.osarchitecture -replace '-bit','';
                 'ProcArchitecture' = $proc.addresswidth
             }
             New-Object -TypeName psobject -Property $properties
         }
     }
 }
`

