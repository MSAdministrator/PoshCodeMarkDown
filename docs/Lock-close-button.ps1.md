---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4059
Published Date: 2013-03-31t15
Archived Date: 2017-05-30t19
---

# lock close button - 

## Description

it’s useful if you offen use alt+space to manipulate host data (copy\paste).

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 $code = @'
 using System;
 using System.Runtime.InteropServices;
 
 namespace CloseButtonToggle {
   internal static class WinAPI {
     [DllImport("kernel32.dll")]
     internal static extern IntPtr GetConsoleWindow();
 
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool DeleteMenu(IntPtr hMenu,
                            uint uPosition, uint uFlags);
 
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool DrawMenuBar(IntPtr hWnd);
 
     [DllImport("user32.dll")]
     internal static extern IntPtr GetSystemMenu(IntPtr hWnd,
                [MarshalAs(UnmanagedType.Bool)]bool bRevert);
 
     const uint SC_CLOSE     = 0xf060;
     const uint MF_BYCOMMAND = 0;
 
     internal static void ChangeCurrentState(bool state) {
       IntPtr hMenu = GetSystemMenu(GetConsoleWindow(), state);
       DeleteMenu(hMenu, SC_CLOSE, MF_BYCOMMAND);
       DrawMenuBar(GetConsoleWindow());
     }
   }
 
   public static class Status {
     public static void Disable() {
       WinAPI.ChangeCurrentState(false); //its 'true' if need to enable
     }
   }
 }
 '@
 
 Add-Type $code
 [CloseButtonToggle.Status]::Disable()
`

