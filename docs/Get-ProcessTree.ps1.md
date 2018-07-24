---
Author: dan jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6362
Published Date: 2016-05-29t09
Archived Date: 2016-07-08t10
---

# get-processtree - 

## Description

original post of gregzakh can be found at https

## Comments



## Usage



## TODO



## function

`get-processtree`

## Code

`#
 #
 function Get-ProcessTree {
   begin {
     Set-Variable ($$ = [Regex].Assembly.GetType(
       'Microsoft.Win32.NativeMethods'
     ).GetMethod('NtQuerySystemInformation')).Name $$
     
     function Get-ProcessChild {
       param(
         [Parameter(Mandatory=$true, Position=0)]
         [PSObject]$Process,
         
         [Parameter(Position=1)]
         [Int32]$Depth = 1
       )
       
       $Processes | Where-Object {
         $_.PPID -eq $Process.PID -and $_.PPID -ne 0
       } | ForEach-Object {
         '{0}{1} ({2})' -f (
           "$([Char]32)" * 2 * $depth
         ), $_.ProcessName, $_.PID
         Get-ProcessChild $_ (++$Depth)
         $Depth--
       }
     }
     
     if (($ta = [PSObject].Assembly.GetType(
       'System.Management.Automation.TypeAccelerators'
     ))::Get.Keys -notcontains 'Marshal') {
       $ta::Add('Marshal', [Runtime.InteropServices.Marshal])
     }
   }
   process {
     try {
       $ret = 0
       $ptr = [Marshal]::AllocHGlobal(1024)
       
       if ($NtQuerySystemInformation.Invoke($null, (
         $par = [Object[]]@(5, $ptr, 1024, $ret)
         $ptr = [Marshal]::ReAllocHGlobal($ptr, [IntPtr]$par[3])
         if (($nts = $NtQuerySystemInformation.Invoke($null, (
           $par = [Object[]]@(5, $ptr, $par[3], 0)
         ))) -ne 0) {
           throw New-Object InvalidOperationException(
             'NTSTATUS: 0x{0:X}' -f $nts
           )
         }
       }
       
       $tmp = $ptr
       $Processes = while (($$ = [Marshal]::ReadInt32($tmp))) {
         New-Object PSObject -Property @{
           ProcessName = [Diagnostics.Process]::GetProcessById((
             $id = [Marshal]::ReadInt32($tmp, 0x44)
           )).Name
           PID = $id
           PPID = [Marshal]::ReadInt32($tmp, 0x48)
         }
         $tmp = [IntPtr]($tmp.ToInt32() + $$)
       }
     }
     catch { $_.Exception }
     finally {
       if ($ptr -ne $null) {
         [Marshal]::FreeHGlobal($ptr)
       }
     }
   }
   end {
     if ($Processes -eq $null) {
       break
     }
     
     $Processes | Where-Object {
       -not (Get-Process -Id $_.PPID -ea 0) -or $_.PPID -eq 0
     } | ForEach-Object {
       '{0} ({1})' -f $_.ProcessName, $_.PID
       Get-ProcessChild $_
     }
     
     [void]$ta::Remove('Marshal')
   }
 }
`

