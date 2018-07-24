---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4548
Published Date: 2015-10-24t11
Archived Date: 2015-07-16t03
---

# looks for parent process - 

## Description

why? why it need use win32_process to check parent process for another process??? there is performancecounter class in the system.diagnostics namespace which can help you determine parent process for a process easily.

## Comments



## Usage



## TODO



## function

`get-parentprocess`

## Code

`#
 #
 function Get-ParentProcess {
   <#
     .LINK
         Follow me on twitter @gregzakharov
         http://msdn.microsoft.com/en-US/library/system.diagnostics.performancecounter.aspx
         Get-Process
   #>
   param(
     [Parameter(Mandatory=$true)]
     [ValidateNotNullOrEmpty()]
     [String]$ProcessName
   )
   
   begin {
     ps | % {$hash = @{}}{
       $pc = New-Object Diagnostics.PerformanceCounter("Process", "Creating Process ID", $_.ProcessName)
       try {
         $ps = [Diagnostics.Process]::GetProcessById($pc.RawValue)
         $hash[$pc.InstanceName] = $pc.RawValue
       }
       catch {
         $hash[$pc.InstanceName] = -1
       }
     }
   }
   process {
     [Diagnostics.Process]::GetProcessesByName($ProcessName) | % {
       if ($hash[$_.ProcessName] -ne -1) {
         foreach ($p in @(ps -id $hash[$ProcessName])){
           '"{0}" (PID {1}) is parent for "{2}" (PID {3})"' -f $p.ProcessName, $p.Id, $_.ProcessName, $_.Id
         }
       }
       else { Write-Host `"$ProcessName`" has not parent process. -fo cyan}
     }
   }
 }
`

