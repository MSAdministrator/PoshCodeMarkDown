---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.1
Encoding: ascii
License: cc0
PoshCode ID: 6631
Published Date: 2017-11-22t07
Archived Date: 2017-03-15t06
---

# get-remoteregistry - 

## Description

a script to do a query on a remote registry key or value (fixed a bug in 2.0)

## Comments



## Usage



## TODO



## function

`get-remoteregistry`

## Code

`#
 #
 ########################################################################################
 ########################################################################################
 ##
 ##
 ##
 ########################################################################################
 param(
     [string]$computer = $(Read-Host "Remote Computer Name")
    ,[string]$Path     = $(Read-Host "Remote Registry Path (must start with HKLM,HKCU,etc)")
    ,[string[]]$Properties
    ,[switch]$Verbose
 )
 
    $root, $last = $Path.Split("\")
    $last = $last[-1]
    $Path = $Path.Substring($root.Length + 1,$Path.Length - ( $last.Length + $root.Length + 2))
    $root = $root.TrimEnd(":")
 
    switch($root) {
       "HKCR"  { $root = "ClassesRoot"}
       "HKCU"  { $root = "CurrentUser" }
       "HKLM"  { $root = "LocalMachine" }
       "HKU"   { $root = "Users" }
       "HKPD"  { $root = "PerformanceData"}
       "HKCC"  { $root = "CurrentConfig"}
       "HKDD"  { $root = "DynData"}
       default { return "Path argument is not valid" }
    }
 
 
    Write-Verbose "Accessing $root from $computer"
    $rootkey = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey($root,$computer)
    if(-not $rootkey) { Write-Error "Can't open the remote $root registry hive" }
 
    Write-Verbose "Opening $Path"
    $key = $rootkey.OpenSubKey( $Path )
    if(-not $key) { Write-Error "Can't open $($root + '\' + $Path) on $computer" }
 
    $subkey = $key.OpenSubKey( $last )
    
    $output = new-object object
 
    if($subkey -and $Properties -and $Properties.Count) {
       foreach($property in $Properties) {
          Add-Member -InputObject $output -Type NoteProperty -Name $property -Value $subkey.GetValue($property)
       }
       Write-Output $output
    } elseif($subkey) {
       Add-Member -InputObject $output -Type NoteProperty -Name "Subkeys" -Value @($subkey.GetSubKeyNames())
       foreach($property in $subkey.GetValueNames()) {
          Add-Member -InputObject $output -Type NoteProperty -Name $property -Value $subkey.GetValue($property)
       }
       Write-Output $output
    }
    else
    {
       $key.GetValue($last)
    }
`

