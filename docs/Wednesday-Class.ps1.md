---
Author: don jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5433
Published Date: 2015-09-17t13
Archived Date: 2015-07-31t01
---

# wednesday class - 

## Description

wednesday elsa module

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
     param(
         [string[]]$ComputerName
     )
 
     foreach ($computer in $ComputerName) {
 
         $services = Get-CimInstance -ClassName Win32_Service -ComputerName $computer -Filter "state='running'"
         foreach ($service in $services) {
 
             $process = Get-CimInstance -ClassName Win32_process `
                                        -ComputerName $computer `
                                        -Filter "processid=$($service.processid)"
 
             $properties = @{'ComputerName'=$Computer;
                             'ProcessID'=$process.ProcessId;
                             'ServiceName'=$service.Name}
             $obj = New-Object -TypeName PSObject -Property $properties
             Write-Output $obj
 
`

