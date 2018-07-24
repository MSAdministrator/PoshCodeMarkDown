---
Author: whertzing56
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5868
Published Date: 2015-05-24t14
Archived Date: 2015-05-29t10
---

# compare reg keys - 

## Description

two functions get-allregkey and compare-allregkey which will recursivly retrieve a key’s properties and subkeys, and their properties and subkeys, as an array of objects, across multiple computers. the compare-allregkey function uses compare-object to compare the arrays returned from each computer to an array returned from a specified reference computer.

## Comments



## Usage



## TODO



## script

`get-allregkey`

## Code

`#
 #
 <#
 .SYNOPSIS 
 Compares Registry Key Properties and subkeys across multiple computers
 .DESCRIPTION
 The function Get-AllRegKey  will recurse down from a given key, returning an array having
     the key's properties, subkeys, and their properties and subkeys.
 Provide Get-AllRegKey a list of computernames, and it will remote to those computers
     and return the properties, etc. of the same key on the remote computer
 Provide function Compare-AllRegKey with the name of the reference computer, a list of
     two or more computer names, and it will call Get-AllRegKey to retrieve the key 
     information from all the listed computers, then use Compare-Object to return 
     just the differences.
 If you want more control over the Compare-Object step, you should modify the 
     function (suggestions welcome for an efficient/concise way to add Compare-Object 
     parameters to the Compare-AllRegKey function)
   
 .PARAMETER ComputerNames
 In the Get-AllRegKey function this is a single computer name or an array of computer names
 In the Compare-AllRegKey function, this must be an array of at least two computer names
 .PARAMETER RegKey
 a single registry key/hive from which recursion starts, using the Registry Provider syntax
 The value defaults to the current-scoped variable $DefaultRegistryKey
 .PARAMETER -ReferenceObject
 Applies only to the Compare-AllRegKey function, this parameter identifies the computer
 against which the other computers are compared. This string must be one of the computer
 names found in the ComputerNames parameter.
 .EXAMPLE
 C:\PS> Get-AllRegKey
 .EXAMPLE
 C:\PS> Get-AllRegKey -RegistryKey 'HKLM:\SOFTWARE\Microsoft\PowerShell'
 .EXAMPLE
 C:\PS> Get-AllRegKey -ComputerNames @('localhost','RemoteCN')
 .EXAMPLE
 C:\PS> Get-AllRegKey -ComputerNames @('localhost','Computer1','Computer2')
 .EXAMPLE
 C:\PS> Get-AllRegKey localhost 'HKLM:\SOFTWARE\Microsoft\PowerShell'
 .EXAMPLE
 C:\PS> Compare-AllRegKey -ComputerNames @('localhost','RemoteCN') 
 .EXAMPLE
 C:\PS> Compare-AllRegKey -ComputerNames @('CN1','CN2') -ReferenceObject CN2
 #>
 
 ################################################################################
 $DefaultRegistryKey='HKLM:\SOFTWARE\Microsoft\PowerShell'
 $computerNames= @('localhost','ncat099')
 
 ################################################################################
 $_getRegKeySB = {Param($RegistryHive)
   function _getRegKey {
     Param($RegistryHive)
     $data=@($RegistryHive)
     $props = Get-ItemProperty -Path $RegistryHive | 
       Select * -Exclude PS*Path,PSChildName,PSDrive,PSProvider
     if ($props) {
       $props = $props | get-member -memberType NoteProperty
       foreach ($p in $props) {$data+=("$RegistryHive`:"+$p.Definition)}
     }
     foreach ($sk in (get-item $RegistryHive).GetSubKeyNames()) {
       $data += (&_getRegKey (($RegistryHive)+'\'+ $sk))
     }
     $data
   }
   &_getRegKey $RegistryHive
 }
 
 ################################################################################
 function Get-AllRegKey {
   Param (
     $computerNames = $computerNames
     ,$RegistryKey = $DefaultRegistryKey
   )
   $AllRegKey = @{}
   foreach ($cn in $computerNames) {
     switch ($cn) {
       {$_ -match "localhost|" + (hostname)} {
         $AllRegKey.$cn = &$_getRegKeySB $RegistryKey
         break
       }
       default {
         $AllRegKey.$cn = (invoke-command -Scriptblock $_getRegKeySB `
             -ArgumentList $RegistryKey -computername "$cn")
       }
     }
   }
   $AllRegKey
 }
 
 ################################################################################
 function Compare-AllRegKey {
   Param (
     $computerNames = $computerNames
     ,$RegistryKey = $DefaultRegistryKey
     ,$ReferenceObject
   )
 Begin {
   if (!$ReferenceObject) {$ReferenceObject =$computerNames[0]}
   else {
     if (!($computerNames -contains $ReferenceObject)) {
           throw ("{0} is not a member of the list {1}" -f $ReferenceObject, `
                ($computerNames -join ','))}
   }
   $AllRegKey = Get-AllRegKey $computerNames $RegistryKey
 } 
 Process {
   $diff = @{};
   foreach ($cn in $computerNames | Where {$cn -ne $ReferenceObject}) {
     $diff.$cn = Compare-Object -ReferenceObject $AllRegKey.$ReferenceObject `
         $AllRegKey.$cn | Where-Object { `
             ($_.SideIndicator -eq '=>') -or ($_.SideIndicator -eq '<=') }
   }
   $diff
 }
 }
 
 Compare-AllRegKey
`

