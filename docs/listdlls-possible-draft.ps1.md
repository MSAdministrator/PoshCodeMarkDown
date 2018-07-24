---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4803
Published Date: 2014-01-16t18
Archived Date: 2014-01-21t08
---

# listdlls(possible draft) - 

## Description

incompleted

## Comments



## Usage



## TODO



## function

`get-dlllist`

## Code

`#
 #
 Set-Alias listdlls Get-DllList
 
 function Get-DllList {
   [CmdletBinding()]
   param(
     [Parameter(Position=0)]
     [Alias('n')]
     [String]$ProcessName,
     
     [Parameter(Position=0)]
     [Alias('id')]
     [String]$ProcessID,
     
     [Parameter(Position=0)]
     [Alias('d')]
     [String]$DllName,
     
     [Alias('v')]
     [Switch]$Version
   )
   
   begin {
     $del = '-' * 75
     
     function tbl([Diagnostics.ProcessModule[]]$modules) {
       $res = $modules | select @{Name='Base'; Expression={'0x' + `
       $_.BaseAddress.ToString('x8')}},@{Name='Size'; Expression={'0x' + `
       $_.ModuleMemorySize.ToString('x')}},@{Name='Path'; Expression={$_.ModuleName}}
       
       return ($res | ft -a)
     }
     
     function get([Diagnostics.Process[]]$scope) {
       $scope | % {
         if ($_.MainModule -ne $null) {
           "`n{0} pid: {1}`n$del" -f $_.MainModule.ModuleName.ToLower(), $_.Id
           tbl $_.Modules
         }
       }
     }
   }
   process {
     if (![String]::IsNullOrEmpty($ProcessName)) {
       get ([Diagnostics.Process]::GetProcessesByName($ProcessName))
     }
     elseif ($ProcessID -ge 0) {
       $p = [Diagnostics.Process]::GetProcessById($ProcessID)
       "`n{0} pid: {1}`n$del" -f $_.MainModule.ModuleName.ToLower(), $_.Id
       tbl $_.Modules
     }
     elseif (![String]::IsNullOrEmpty($DllName)) {
       if ($Version) {
         $isit = (gcm -ea 0 -c Application $DllName).FileVersionInfo | fl
         
         if ($isit -eq $null) {
           Write-Host Can not find module directly. -fo Red
         }
         else {$isit}
       }
       
       [Diagnostics.Process]::GetProcesses() | % {
         if ($_.Modules -match $DllName) {
           '{0, 25} pid: {1}' -f $_.ProcessName, $_.Id
         }
       }
     }
     else {
       get ([Diagnostics.Process]::GetProcesses())
     }
   }
   end {}
 }
`

