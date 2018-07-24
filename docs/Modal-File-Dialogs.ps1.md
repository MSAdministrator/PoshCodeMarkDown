---
Author: george howarth
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2002
Published Date: 2011-07-22t02
Archived Date: 2017-03-25t01
---

# modal file dialogs - 

## Description

there are problems with displaying modal dialogs from powershell in xp sp3. when the showdialog() method is called, the dialog is not modal and is behind the powershell window. to solve this problem, you need to add a class that implements system.windows.forms.iwin32window and instantiate that class with the handle to the main window of the running process, and then pass that handle as a parameter to the showdialog() method to make the dialog act modally.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 Add-Type -TypeDefinition @"
 using System;
 using System.Windows.Forms;
 
 public class Win32Window : IWin32Window
 {
     private IntPtr _hWnd;
     
     public Win32Window(IntPtr handle)
     {
         _hWnd = handle;
     }
 
     public IntPtr Handle
     {
         get { return _hWnd; }
     }
 }
 "@ -ReferencedAssemblies "System.Windows.Forms.dll"
 
 $owner = New-Object Win32Window -ArgumentList ([System.Diagnostics.Process]::GetCurrentProcess().MainWindowHandle)
 $dialog = New-Object System.Windows.Forms.OpenFileDialog
 $dialog.ShowDialog($owner)
`

