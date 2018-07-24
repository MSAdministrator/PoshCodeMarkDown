---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1865
Published Date: 
Archived Date: 2010-05-24t16
---

# set-opacity - 

## Description

like set-blur, except it runs on windows xp … you know … without the blur

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
    #.Synopsis
    #.Parameter color
    #.Parameter opacity
    #.Parameter Off
    #.Parameter Handle
    #.Example 
    #
    #.Example 
    #
    param([byte]$opacity, [System.Drawing.Color]$color, [switch]$Off, [IntPtr]$handle = (ps -Id $pid).MainWindowHandle)
    
 
 Add-Type -Type @"
 using System;
 using System.Runtime.InteropServices;
 namespace Huddled {
    public class Opacity {
    
       [DllImport("user32.dll")]
       public static extern int SetWindowLong(IntPtr hWnd, int nIndex, int dwNewLong);
       
       [DllImport("user32.dll", SetLastError=true)]
       public static extern int GetWindowLong(IntPtr hWnd, int nIndex);
 
       [DllImport("user32.dll")]
       public static extern bool SetLayeredWindowAttributes(IntPtr hwnd, uint crKey, byte bAlpha, uint dwFlags);
    }
    
    [StructLayout(LayoutKind.Sequential)]
    public struct ColorRef
    {
       public uint ColorDWORD;
 
       public static implicit operator uint(ColorRef cr) {
          return cr.ColorDWORD;
       }
       public static implicit operator ColorRef(uint crKey) {
          ColorRef cr = new ColorRef();
          cr.ColorDWORD = crKey;
          return cr;
       }
       
       public static implicit operator System.Drawing.Color(ColorRef cr){
          return System.Drawing.Color.FromArgb((int)(0x000000FFU & cr.ColorDWORD), (int)(0x0000FF00U & cr.ColorDWORD) >> 8, (int)(0x00FF0000U & cr.ColorDWORD) >> 16);
       }
       
       public static implicit operator ColorRef(System.Drawing.Color color){
          ColorRef cr = new ColorRef();
          cr.ColorDWORD = (uint)color.R + (((uint)color.G) << 8) + (((uint)color.B) << 16);
          return cr;
       }
    }
 }
 "@ -Ref System.Drawing
 
 function global:Set-Opacity {
    #.Synopsis
    #.Parameter color
    #.Parameter opacity
    #.Parameter Off
    #.Parameter Handle
    #.Example 
    #
    #.Example 
    #
    param([byte]$opacity, [System.Drawing.Color]$color, [switch]$Off, [IntPtr]$handle = (ps -Id $pid).MainWindowHandle)
    
    if($Off) { 
       $style = [Huddled.Opacity]::GetWindowLong($handle, -20) -bor 0x80000 -bxor 0x80000
    } else {
       $style = [Huddled.Opacity]::GetWindowLong($handle, -20) -bor 0x80000      
    }
    $style = [Huddled.Opacity]::SetWindowLong($Handle, -20, $style);
 
    $flag = 0
    if($color) { $flag += 1 } else { $color = [System.Drawing.Color]::Black }
    if($opacity) { $flag += 2 } else { $opacity = 255 }
    
    $null = [Huddled.Opacity]::SetLayeredWindowAttributes($handle, ([uInt32][Huddled.ColorRef]$color), $opacity, $flag)
 }
 
 Set-Opacity @PSBoundParameters
`

