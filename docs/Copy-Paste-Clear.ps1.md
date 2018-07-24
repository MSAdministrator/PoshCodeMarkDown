---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4086
Published Date: 2013-04-08t14
Archived Date: 2016-10-20t05
---

# copy\paste\clear - 

## Description

this console application demonstrates pinvoke way to work with buffer.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 using System;
 using System.Text;
 using System.Threading;
 using System.Reflection;
 using System.Runtime.InteropServices;
 
 [assembly: AssemblyVersion("2.0.0.0")]
 
 namespace Clip {
   internal static class WinAPI {
     [DllImport("kernel32.dll")]
     internal static extern IntPtr GetConsoleWindow();
 
     [DllImport("kernel32.dll")]
     internal static extern IntPtr GlobalAlloc(uint uFlags, UIntPtr dwBytes);
 
     [DllImport("kernel32.dll")]
     internal static extern IntPtr GlobalLock(IntPtr hMem);
 
     [DllImport("kernel32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool GlobalUnlock(IntPtr hMem);
 
     [DllImport("kernel32.dll", BestFitMapping = false, ThrowOnUnmappableChar = true)]
     internal static extern IntPtr lstrcpy(IntPtr lpString1, string lpString2);
 
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool CloseClipboard();
 
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool EmptyClipboard();
 
     [DllImport("user32.dll")]
     internal static extern IntPtr GetOpenClipboardWindow();
 
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool OpenClipboard(IntPtr hWndNewOwner);
 
     [DllImport("user32.dll")]
     internal static extern IntPtr SendMessage(IntPtr hWnd, uint Msg,
                                       IntPtr wParam, IntPtr lParam);
 
     [DllImport("user32.dll")]
     internal static extern IntPtr SetClipboardData(uint uFormat, IntPtr hMem);
 
     internal static void CopyToClipboard(string text) {
       IntPtr mem = GlobalAlloc((uint)0x0042, (UIntPtr)(text.Length + 1)); //GHND = 0x0042;
       IntPtr lck = GlobalLock(mem);
 
       lstrcpy(lck, text);
       GlobalUnlock(mem);
       OpenClipboard(GetOpenClipboardWindow());
       EmptyClipboard();
       SetClipboardData((uint)1, mem); //CF_TEXT = 1;
       CloseClipboard();
     }
 
     internal static void PasteFromClipboard() {
       IntPtr hndl = GetConsoleWindow();
       SendMessage(hndl, 0x0111, (IntPtr)0xfff1, IntPtr.Zero); //WM_COMMAND = 0x0111, 0xfff1 is console pointer
     }
 
     internal static void ClearClipboardData() {
       OpenClipboard(GetOpenClipboardWindow());
       EmptyClipboard();
       CloseClipboard();
     }
   }
 
   internal sealed class Program {
     static void Main() {
       StringBuilder sb = new StringBuilder();
 
       for (int i = 0; i < 10; i++)
         sb.Append("test, ");
 
       //copy to clipboard test
       WinAPI.CopyToClipboard(sb.ToString());
       Thread.Sleep(1500);
       //paste to console text test
       WinAPI.PasteFromClipboard();
       Thread.Sleep(1500);
       //clear clipboard test
       WinAPI.ClearClipboardData();
     }
   }
 }
`

