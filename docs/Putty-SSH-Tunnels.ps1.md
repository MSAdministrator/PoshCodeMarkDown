---
Author: crazydave
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6546
Published Date: 2016-10-04t18
Archived Date: 2016-10-29t00
---

# putty ssh tunnels - 

## Description

two simple functions to get and set the ssh tunnels that putty will establish

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
  
 function AddPortForward {
     param(
         [String] $Connection,
         [UInt16] $SrcPort,
         [String] $DstHost,
         [UInt16] $DstPort
     )
     $baseSessionFolder = 'Software\SimonTatham\PuTTY\Sessions'
     $key = [Microsoft.Win32.Registry]::CurrentUser.OpenSubKey($baseSessionFolder + "\" + $Connection, $true)
     $currentValue = $key.GetValue("PortForwardings")
     $newValue = $currentValue + "L" + $SrcPort + "=" + $DstHost + ":" + $DstPort + ","
     $key.SetValue("PortForwardings", $newValue, [Microsoft.Win32.RegistryValueKind]::String)
  }
 
 function GetPortForwards {
     $baseSessionFolder = 'HKCU:\Software\SimonTatham\PuTTY\Sessions'
     gci $baseSessionFolder | % {
     $key = $_
     $key.GetValue("PortForwardings").Split(',', [System.StringSplitOptions]::RemoveEmptyEntries) | % {
         $parts = $_.Split('=')
         New-Object PSObject -Property @{
             Connection = $key.PSChildName
             SRC = $parts[0]
             DST = $parts[1]
         }
     }
  }
 }
`

