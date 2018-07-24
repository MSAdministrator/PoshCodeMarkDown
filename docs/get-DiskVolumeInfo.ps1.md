---
Author: richard vantreas
Publisher: 
Copyright: 
Email: 
Version: 5.2
Encoding: ascii
License: cc0
PoshCode ID: 2977
Published Date: 2012-09-28t11
Archived Date: 2012-01-14t07
---

# get-diskvolumeinfo - 

## Description

this function returns information about disk volumes, works on win2k, winxp, win2003, win2008, winvista, win7.  can be used in a pipeline with -raw switch or stand-alone.

## Comments



## Usage



## TODO



## function

`get-diskvolumeinfo`

## Code

`#
 #
 function get-DiskVolumeInfo
 {
     <#
         .SYNOPSIS
         Returns information about disk volumes including freespace
         
         .DESCRIPTION
         Returns information about disk volumes including freespace
         
         .EXAMPLE
         show-InnerException ExceptionObject
         Shows the inner exception object of the error object that is passed to the
         function.
         
         .Notes
     	ChangeLog    :
     	Date	    Initials	Short Description
         02/18/2009  RLV         New
  
         .Link
         http://msdn.microsoft.com/en-us/library/aa394239(v=VS.85).aspx
         
         .Link
         http://msdn.microsoft.com/en-us/library/aa394515(VS.85).aspx
         
         .Link
         http://msdn.microsoft.com/en-us/library/windows/desktop/aa394173(v=vs.85).aspx
     #>
 
     [CmdletBinding()]
     param
         (
             [Parameter(Mandatory=$true)][string]$ComputerName,
             [Parameter(Mandatory=$false)][switch]$Raw
         )
         
     trap 
     {
         show-InnerException -ex $_
         continue
     }
 
     write-verbose "$($MyInvocation.InvocationName) - Begin function"
     foreach($Key in $PSBoundParameters.Keys){ write-verbose "$($MyInvocation.InvocationName) PARAM: -$Key - $($PSBoundParameters[$Key])" }
 
     $VolArray = @()
     
     if($(gwmi win32_operatingSystem -comp $ComputerName).version -ge '5.2')
     {
         $VolArray = gwmi win32_volume -Computer $ComputerName | Select-Object `
             @{Name='Computer';Expression={$ComputerName}}, `
             @{Name='VolumeName';Expression={if($_.Name -like "\\?\Volume*"){'\\?\Volume'}else{$_.Name}}}, `
             @{Name='Capacity_GB';Expression={[math]::Round($_.Capacity/1GB)}}, `
             @{Name='FreeSpace_GB';Expression={[math]::Round($_.FreeSpace/1GB)}}, `
             @{Name='Pct_Free';Expression={if($_.Capacity -gt 0){[math]::Round(($_.FreeSpace/$_.Capacity)*100)}else{0}}}, `
             @{Name='BlockSize_KB';Expression={[math]::Round($_.Blocksize/1KB)}}
     }
     else
     {
         $VolArray = gwmi win32_LogicalDisk -Computer $ComputerName | Select-Object `
             @{Name='Computer';Expression={$ComputerName}}, `
             @{Name='VolumeName';Expression={$_.Name}}, `
             @{Name='Capacity_GB';Expression={[math]::Round($_.Size/1GB)}}, `
             @{Name='FreeSpace_GB';Expression={[math]::Round($_.FreeSpace/1GB)}}, `
             @{Name='Pct_Free';Expression={if($_.Size -gt 0){[math]::Round(($_.FreeSpace/$_.Size)*100)}else{0}}}
    }
     
     if($Raw){  $VolArray   }
     else{   $VolArray | ft -auto    }
     
     write-verbose "$($MyInvocation.InvocationName) - End function"
 }
`

