---
Author: mark schill
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 566
Published Date: 2009-09-03t15
Archived Date: 2016-03-17t00
---

# remove-trash - 

## Description

a simple script cmdlet that allows you to empty your recycle bin.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 
 add-type @"
 using System;
 using System.Runtime.InteropServices;
 using Microsoft.Win32;
 
 namespace recyclebin
 {
     public class Empty
     {
         [DllImport("shell32.dll")]
         static extern int SHEmptyRecycleBin(IntPtr hWnd, string pszRootPath, uint dwFlags);
 
         public static void EmptyTrash(string RootDrive, uint Flags)
         {
            SHEmptyRecycleBin(IntPtr.Zero, RootDrive, Flags);
         }
     }
 }
 "@
 
 cmdlet Remove-Trash -snapin CMSchill.RemoveTrash
 {
 	param(
 	[Parameter(Position=0, Mandatory=$false)][string]$Drive = "",
 	[switch]$NoConfirmation,
 	[switch]$NoProgressGui,
 	[switch]$NoSound
 	)
 	[int]$Flags = 0
 	if ( $NoConfirmation ) { $Flags = $Flags -bor 1 }
 	if ( $NoProgressGui ) { $Flags = $Flags -bor 2 }
 	if ( $NoSound ) { $Flags = $Flags -bor 4 }
 
 	[recyclebin.Empty]::EmptyTrash($RootPath, $Flags)
 }
`

