---
Author: mnaoumov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3537
Published Date: 2014-07-23t18
Archived Date: 2014-05-13t20
---

# $env - 

## Description

add directory to environment path variable permanently. apply changes immediately without requiring reboot.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 
 param(
     [string] $AddedFolder,
     [bool] $ApplyImmediately = $true
 )
 
 $environmentRegistryKey = 'Registry::HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\Environment'
 
 $oldPath = (Get-ItemProperty -Path $environmentRegistryKey -Name PATH).Path
 
 
 if (!$AddedFolder)
 {
     Write-Warning 'No Folder Supplied. $ENV:PATH Unchanged'
     return
 }
 
 if ($ENV:PATH | Select-String -SimpleMatch $AddedFolder)
 {
     Write-Warning 'Folder already within $ENV:PATH'
     return
 }
 
 $newPath = $oldPath + �;� + $AddedFolder
 
 Set-ItemProperty -Path $environmentRegistryKey -Name PATH -Value $newPath
 
 if ($ApplyImmediately)
 {
     if (-not ("Win32.NativeMethods" -as [Type]))
     {
         Add-Type -Namespace Win32 -Name NativeMethods -MemberDefinition @"
     [DllImport("user32.dll", SetLastError = true, CharSet = CharSet.Auto)]
     public static extern IntPtr SendMessageTimeout(
         IntPtr hWnd, uint Msg, UIntPtr wParam, string lParam,
         uint fuFlags, uint uTimeout, out UIntPtr lpdwResult);
 "@
     }
 
     $HWND_BROADCAST = [IntPtr] 0xffff;
     $WM_SETTINGCHANGE = 0x1a;
     $result = [UIntPtr]::Zero
 
     [Win32.Nativemethods]::SendMessageTimeout($HWND_BROADCAST, $WM_SETTINGCHANGE, [UIntPtr]::Zero, "Environment", 2, 5000, [ref] $result);
 }
`

