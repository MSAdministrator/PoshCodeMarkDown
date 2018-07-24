---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2196
Published Date: 2011-09-09t21
Archived Date: 2017-05-16t12
---

# move-lockedfile.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Registers a locked file to be moved at the next system restart.
 
 .EXAMPLE
 
 Move-LockedFile c:\temp\locked.txt c:\temp\locked.txt.bak
 
 #>
 
 param(
     $Path,
 
     $Destination
 )
 
 Set-StrictMode -Version Latest
 
 $path = (Resolve-Path $path).Path
 $destination = $executionContext.SessionState.`
     Path.GetUnresolvedProviderPathFromPSPath($destination)
 
 $MOVEFILE_DELAY_UNTIL_REBOOT = 0x00000004
 $memberDefinition = @'
 [DllImport("kernel32.dll", SetLastError=true, CharSet=CharSet.Auto)]
 public static extern bool MoveFileEx(
     string lpExistingFileName, string lpNewFileName, int dwFlags);
 '@
 $type = Add-Type -Name MoveFileUtils `
     -MemberDefinition $memberDefinition -PassThru
 
 $type::MoveFileEx($path, $destination, $MOVEFILE_DELAY_UNTIL_REBOOT)
`

