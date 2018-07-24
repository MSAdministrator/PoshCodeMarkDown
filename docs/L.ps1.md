---
Author: mtown_nerd
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4788
Published Date: 2014-01-11t09
Archived Date: 2014-01-15t19
---

# l - 

## Description

a simple function for connecting a unc path to a specified windows drive letter.  some other things i’d like to see added

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function MapNetDrive 
 {
     param(
     
     #
     [Parameter(Position=0,Mandatory=$true)]
     [string]$DriveLetter="Z:",
     [Parameter(Position=1,Mandatory=$true)]
     [string]$Path,
     
     #
     [Parameter(Position=2,Mandatory=$false)]
     [switch]$Credentials,
     [Parameter(Position=3,Mandatory=$false)]
     [switch]$Stay,
     [Parameter(Position=4,Mandatory=$false)]
     [switch]$Force
     )
     
     #
     $map=New-Object -ComObject WScript.Network
     
     #
     if($map.EnumNetworkDrives() -contains $DriveLetter) 
     {
         if(!$Force) 
         {throw "Can't map $driveLetter because it's already mapped.  Use -Force to override."}
         
         try 
         {$map.RemoveNetworkDrive($DriveLetter,$Force.ToBool(),$Stay.ToBool())}
             catch 
             {
             Write-Error -Exception $_.Exception.InnerException -Message "Error removing '$driveLetter'
                 $($_.Exception.InnerException.InnerException.Message)"
             }
     }   
     
     #
     #"$Stay" is placeholder for "bUpdateProfile" argument of the MapNetworkDrive() method, which determines whether the new
     if($Credentials) 
     {
        [System.Management.Automation.PSCredential]$creds=$(Get-Credential) #-Credential $($(Get-WmiObject -Class Win32_ComputerSystem).UserName)
        $map.MapNetworkDrive($DriveLetter,$Path,$Stay.ToBool(),$creds.UserName,$creds.GetNetworkCredential().Password)
     } 
         else {$map.MapNetworkDrive($DriveLetter,$Path,$Stay.ToBool())} 
        
     #
     $explore=New-Object -ComObject Shell.Application
     $explore.Open($DriveLetter)
 }
`

