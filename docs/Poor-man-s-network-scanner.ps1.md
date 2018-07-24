---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4429
Published Date: 
Archived Date: 2013-09-04t07
---

#  - 

## Description

poor man’s network scanner

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 
 $code = @"
 using System;
 using System.Drawing;
 using System.Runtime.InteropServices;
 
 namespace System
 {
 	public class IconExtractor
 	{
 
 	 public static Icon Extract(string file, int number, bool largeIcon)
 	 {
 	  IntPtr large;
 	  IntPtr small;
 	  ExtractIconEx(file, number, out large, out small, 1);
 	  try
 	  {
 	   return Icon.FromHandle(largeIcon ? large : small);
 	  }
 	  catch
 	  {
 	   return null;
 	  }
 
 	 }
 	 [DllImport("Shell32.dll", EntryPoint = "ExtractIconExW", CharSet = CharSet.Unicode, ExactSpelling = true, CallingConvention = CallingConvention.StdCall)]
 	 private static extern int ExtractIconEx(string sFile, int iIndex, out IntPtr piLargeVersion, out IntPtr piSmallVersion, int amountIcons);
 
 	}
 }
 "@
 
 
 
 Add-Type -TypeDefinition $code -ReferencedAssemblies System.Drawing
 
 [void][System.Reflection.Assembly]::LoadWithPartialName("System.windows.forms")
  
 $form1 = New-Object System.Windows.Forms.form
 $NotifyIcon= New-Object System.Windows.Forms.NotifyIcon
 $ContextMenu = New-Object System.Windows.Forms.ContextMenu
 $MenuItem = New-Object System.Windows.Forms.MenuItem
 $nmapTimer = New-Object System.Windows.Forms.Timer
 $TrayIcon = [System.IconExtractor]::Extract("shell32.dll", 18, $true)
 
 $IP1 = "192.168.0.180"
 $script:IP1r = 0
 
 $form1.ShowInTaskbar = $false
 $form1.WindowState = "minimized"
 
  
 $NotifyIcon.Icon =  $TrayIcon
 $NotifyIcon.ContextMenu = $ContextMenu
 $NotifyIcon.contextMenu.MenuItems.AddRange($MenuItem)
 $NotifyIcon.Visible = $True
 
  
 $nmapTimer.add_Tick({nmaptest})
 $nmapTimer.start()
 
  
 $MenuItem.Text = "Exit"
 $MenuItem.add_Click({
 
 	$nmapTimer.stop()
 	$NotifyIcon.Visible = $False
 
 	$form1.close()
 
 })
 
  
 
 function nmaprun ($id) {
 
     return ( [bool](nmap -sP $id|select-string 'Host is up' ) )
 
 }
 
  
 function nmaptest {
 
 
     $NotifyIcon.Text = "IP scanner del pover'uomo"
     $IP1s = nmaprun $IP1
     
     if ( $IP1s ) {
             if ( ! ( $script:IP1r ) ) {
                 $NotifyIcon.ShowBalloonTip(5000,"Risultato scansione", ($IP1 + " è raggiungibile"),[system.windows.forms.ToolTipIcon]"Warning")
                 $script:IP1r=1
             }
         }
         else {
             if ( $script:IP1r ) {
                 $NotifyIcon.ShowBalloonTip(5000,"Risultato scansione", ($IP1 + " non è più raggiungibile"),[system.windows.forms.ToolTipIcon]"Error")
                 $script:IP1r=0
             }
         }
 
 }
 
 nmaptest
 [void][System.Windows.Forms.Application]::Run($form1)
`

