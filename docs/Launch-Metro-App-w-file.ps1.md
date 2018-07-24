---
Author: bob p
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6479
Published Date: 2016-08-19t02
Archived Date: 2016-08-21t07
---

# launch metro app w/ file - 

## Description

this script can be run with powershell to launch a metro application against a file. it is really intended to be used from a batch file, which is embedded as a comment within the script.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 Created by: Bob P
 Version: 1.0
 Date: 08/18/16
  
 This script file is used to run a metro app from a command line (e.g. CMD)
 
 Usage: 
 PowerShell.exe -ExecutionPolicy Bypass -Command "& '_path_to_this_file'" _execution_string _filename_
 
 For example, if you:
 * Save this file as c:\utils\Run4.ps1
 * Want to launch Movies and TV
 * Want that program to display a file at c:\users\user1\MyMovie.mp4, you would have this command line:
 PowerShell.exe -ExecutionPolicy Bypass -Command "& 'c:\utils\Run4.ps1'" Microsoft.ZuneVideo_8wekyb3d8bbwe!Microsoft.ZuneVideo c:\users\user1\MyMovie.mp4
 
 Here is a batch file that will launch Movies & TV:
 ---- Start LaunchMovies.cmd
 @echo off
 if *%1*==** goto badArg
 for %%F in (%1) do set Q='%%~fF'
 PowerShell.exe -ExecutionPolicy Bypass -Command "& 'c:\utils\run4.ps1'" Microsoft.ZuneVideo_8wekyb3d8bbwe!Microsoft.ZuneVideo %Q%
 goto done
 
 :badArg
 echo Usage: %0 File-to-Play
 goto done
 
 :done
 ---- End LaunchMovies.cmd
 
 Thanks to http://poshcode.org/3740 and http://stackoverflow.com/questions/12925748/iapplicationactivationmanageractivateapplication-in-c/
 #>
 
 $code = @'
 using System;
 using System.Runtime.CompilerServices;
 using System.Runtime.InteropServices;
 
 namespace StartMetroApp {
     public enum ActivateOptions
     {
         None = 0x00000000,  // No flags set
         DesignMode = 0x00000001,  // The application is being activated for design mode, and thus will not be able to
         // to create an immersive window. Window creation must be done by design tools which
         // load the necessary components by communicating with a designer-specified service on
         // the site chain established on the activation manager.  The splash screen normally
         // shown when an application is activated will also not appear.  Most activations
         // will not use this flag.
         NoErrorUI = 0x00000002,  // Do not show an error dialog if the app fails to activate.                                
         NoSplashScreen = 0x00000004,  // Do not show the splash screen when activating the app.
     }
 
     [ComImport, Guid("2e941141-7f97-4756-ba1d-9decde894a3d"), InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
     interface IApplicationActivationManager
     {
         // Activates the specified immersive application for the "Launch" contract, passing the provided arguments
         // string into the application.  Callers can obtain the process Id of the application instance fulfilling this contract.
         IntPtr ActivateApplication([In] String appUserModelId, [In] String arguments, [In] ActivateOptions options, [Out] out UInt32 processId);
         IntPtr ActivateForFile([In] String appUserModelId, [In] [MarshalAs(UnmanagedType.Interface, IidParameterIndex = 2)] /*IShellItemArray* */ IShellItemArray itemArray, [In] String verb, [Out] out UInt32 processId);
         IntPtr ActivateForProtocol([In] String appUserModelId, [In] IntPtr /* IShellItemArray* */itemArray, [Out] out UInt32 processId);
     }
 
     [ComImport, Guid("45BA127D-10A8-46EA-8AB7-56EA9078943C")]//Application Activation Manager
     public class ApplicationActivationManager : IApplicationActivationManager
     {
         [MethodImpl(MethodImplOptions.InternalCall, MethodCodeType = MethodCodeType.Runtime)/*, PreserveSig*/]
         public extern IntPtr ActivateApplication([In] String appUserModelId, [In] String arguments, [In] ActivateOptions options, [Out] out UInt32 processId);
         [MethodImpl(MethodImplOptions.InternalCall, MethodCodeType = MethodCodeType.Runtime)]
         public extern IntPtr ActivateForFile([In] String appUserModelId, [In] [MarshalAs(UnmanagedType.Interface, IidParameterIndex = 2)]  /*IShellItemArray* */ IShellItemArray itemArray, [In] String verb, [Out] out UInt32 processId);
         [MethodImpl(MethodImplOptions.InternalCall, MethodCodeType = MethodCodeType.Runtime)]
         public extern IntPtr ActivateForProtocol([In] String appUserModelId, [In] IntPtr /* IShellItemArray* */itemArray, [Out] out UInt32 processId);
     }
 
     [ComImport]
     [InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
     [Guid("43826d1e-e718-42ee-bc55-a1e261c37bfe")]
     public interface IShellItem
     {
     }
 
     [ComImport]
     [InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
     [Guid("b63ea76d-1f85-456f-a19c-48159efa858b")]
     public interface IShellItemArray
     {
     }
 
     public static class MainProgram
     {
         [DllImport("shell32.dll", CharSet = CharSet.Unicode, PreserveSig = false)]
         private static extern void SHCreateItemFromParsingName(
             [In] [MarshalAs(UnmanagedType.LPWStr)] string pszPath,
             [In] IntPtr pbc,
             [In] [MarshalAs(UnmanagedType.LPStruct)] Guid iIdIShellItem,
             [Out] [MarshalAs(UnmanagedType.Interface, IidParameterIndex = 2)] out IShellItem iShellItem);
 
         [DllImport("shell32.dll", CharSet = CharSet.Unicode, PreserveSig = false)]
         private static extern void SHCreateShellItemArrayFromShellItem(
             [In] [MarshalAs(UnmanagedType.Interface, IidParameterIndex = 2)] IShellItem psi,
             [In] [MarshalAs(UnmanagedType.LPStruct)] Guid iIdIShellItem,
             [Out] [MarshalAs(UnmanagedType.Interface, IidParameterIndex = 2)] out IShellItemArray iShellItemArray);
 
 
         public static IShellItemArray GetShellItemArray(string sourceFile)
         {
             IShellItem item = GetShellItem(sourceFile);
             IShellItemArray array = GetShellItemArray2(item);
 
             return array;
         }
 
         public static IShellItem GetShellItem(string sourceFile)
         {
             IShellItem iShellItem = null;
             Guid iIdIShellItem = new Guid("43826d1e-e718-42ee-bc55-a1e261c37bfe");
             SHCreateItemFromParsingName(sourceFile, IntPtr.Zero, iIdIShellItem, out iShellItem);
 
             return iShellItem;
         }
 
         public static IShellItemArray GetShellItemArray2(IShellItem shellItem)
         {
             IShellItemArray iShellItemArray = null;
             Guid iIdIShellItemArray = new Guid("b63ea76d-1f85-456f-a19c-48159efa858b");
             SHCreateShellItemArrayFromShellItem(shellItem, iIdIShellItemArray, out iShellItemArray);
 
             return iShellItemArray;
         }
 
         public static void LaunchMetroProgram(string progID, string filePath) {
             ApplicationActivationManager appActiveManager = new ApplicationActivationManager(); 
             uint pid;
             IShellItemArray array = GetShellItemArray(filePath);
    
             appActiveManager.ActivateForFile(progID, array, "Open", out pid);
 
         }
     }
 }
 '@
 
 add-type -TypeDefinition $code
 
 [StartMetroApp.MainProgram]::LaunchMetroProgram($args[0], $args[1]);
`

