---
Author: huajun gu
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3482
Published Date: 2012-06-25t17
Archived Date: 2012-06-30t22
---

# test-vm - 

## Description

this function is prepared for hyper-v 3.0 since we don’t have a native cmdlet in the module.

## Comments



## Usage



## TODO



## function

`test-vm`

## Code

`#
 #
 Function Test-VM
 {
     [cmdletbinding()]
     Param
     (
         [Parameter(Mandatory=$true,Position=1)]
         [string[]]$Name,
         [Parameter(Mandatory=$true,Position=2)]
         [string[]]$ComputerName
     )
     Process
     {
         $results = @()
         foreach ($cName in $ComputerName) {
             foreach ($vName in $Name) {
                 $result = New-Object System.Management.Automation.PSObject
                 Try
                 {
                     $vm = Get-VM -ComputerName $cName -Name $vName -ErrorAction Stop
                 }
                 Catch
                 {
                 }
                 if ($vm -ne $null) {
                     $Existence = $true
                 } else {
                     $Existence = $false
                 }				
                 $result | Add-Member -NotePropertyName ComputerName -NotePropertyValue $cName
                 $result | Add-Member -NotePropertyName Name -NotePropertyValue $vName
                 $result | Add-Member -NotePropertyName Existence -NotePropertyValue $Existence
                 $results += $result
             }
         }
         return $results
     }
 }
`

