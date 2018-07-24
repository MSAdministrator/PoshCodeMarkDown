---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5702
Published Date: 2016-01-21t22
Archived Date: 2016-08-19t05
---

# appxapps - 

## Description

this module provides two functions

## Comments



## Usage



## TODO



## module

`get-appxapp`

## Code

`#
 #
 
 
 
 Add-Type @"
 using System;
 using System.Runtime.CompilerServices;
 using System.Runtime.InteropServices;
 namespace Windows {
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
         IntPtr ActivateForFile([In] String appUserModelId, [In] IntPtr /*IShellItemArray* */ itemArray, [In] String verb, [Out] out UInt32 processId);
         IntPtr ActivateForProtocol([In] String appUserModelId, [In] IntPtr /* IShellItemArray* */itemArray, [Out] out UInt32 processId);
     }
 
     [ComImport, Guid("45BA127D-10A8-46EA-8AB7-56EA9078943C")]//Application Activation Manager
     public class ApplicationActivationManager : IApplicationActivationManager
     {
         [MethodImpl(MethodImplOptions.InternalCall, MethodCodeType = MethodCodeType.Runtime)/*, PreserveSig*/]
         public extern IntPtr ActivateApplication([In] String appUserModelId, [In] String arguments, [In] ActivateOptions options, [Out] out UInt32 processId);
         [MethodImpl(MethodImplOptions.InternalCall, MethodCodeType = MethodCodeType.Runtime)]
         public extern IntPtr ActivateForFile([In] String appUserModelId, [In] IntPtr /*IShellItemArray* */ itemArray, [In] String verb, [Out] out UInt32 processId);
         [MethodImpl(MethodImplOptions.InternalCall, MethodCodeType = MethodCodeType.Runtime)]
         public extern IntPtr ActivateForProtocol([In] String appUserModelId, [In] IntPtr /* IShellItemArray* */itemArray, [Out] out UInt32 processId);
     }
 }
 "@
 
 $Activator = new-object Windows.ApplicationActivationManager
 
 Update-TypeData -TypeName Microsoft.Windows.Appx.PackageManager.Commands.AppxPackage -DefaultDisplayProperty Name -DefaultDisplayPropertySet Name, Version, Publisher -EA 0
 Update-TypeData -TypeName Microsoft.Windows.Appx.Application -DefaultDisplayProperty AppUserModelId -DefaultDisplayPropertySet PackageName, Id, Publisher, Protocols, Extensions -EA 0
 
 function Get-AppxApp {
     #.Synopsis
     #.Example
     #
     #.Example
     #
     #.Example
     #
     param(
         [Parameter(Position=0)]
         $PackageName = "*",
         [Parameter(Position=1)]
         $AppId = "*",
 
         [Parameter(Position=2)]
         $Protocol
     )
     Get-AppXPackage $PackageName -pv Package |
         Get-AppxPackageManifest | % { 
             foreach($Application in $_.Package.Applications.Application) {
                 if($Application.Id -like $AppId) {
                     if($Protocol -and !($Application.Extensions.Extension.Protocol.Name | ? { ($_ + "://") -match (($Protocol -replace '\*','.*') + "(://)?") })) {
                         continue
                     }
 
                     [PSCustomObject]@{
                         PSTypeName = "Microsoft.Windows.Appx.Application"
                         AppUserModelId = $Package.PackageFamilyName + "!" + $Application.Id
                         PackageName = $Package.Name
                         Publisher = $Package.Publisher
                         PublisherId = $Package.PublisherId
                         PackageFamilyName = $Package.PackageFamilyName
                         Id = $Application.Id
                         Protocols = @( $Application.Extensions.Extension.Protocol.Name | %{ $_ + "://" })
                         Extensions = @( $Application.Extensions.Extension.FileTypeAssociation.SupportedFileTypes.FileType )
                         Package = $Package
                     }
                 }
             }
         }
 }
 
 
 function Start-AppxApp {
     #.Synopsis
     #.Example
     #
     #.Example
     #
     [CmdletBinding(DefaultParameterSetName="ByName")]
     param(
         [Parameter(Mandatory=$true, Position=0, ParameterSetName="ByName")]
         [Alias("Name")]
         [string]$PackageName,
 
         [Parameter(Mandatory=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName="PackagePublisherApplicationID")]
         [ValidateScript({if($_ -match ".*_.*!.*") { $true } else { throw "Invalid AppUserModelId. Should be: PackageName_PublisherId!AppId"}})]
         [string]$AppUserModelId,
 
         [Parameter(Mandatory=$true, ParameterSetName="Protocol")]
         [Alias("Protocols")]
         [ValidateScript({if($_ -match "\w+://.*") { $true } else { throw "Invalid Protocol. Should start with: protocol://"}})]
         [string]$Protocol
     )
     if($AppUserModelId) {
         $Null = $Activator.ActivateApplication($AppUserModelId, $null, [Windows.ActivateOptions]::None, [ref]0)
     }
     if($PackageName) {
         $Apps = @(Get-AppxApp -PackageName $PackageName)
         if($Apps.Count -gt 1) {
             Write-Warning "'$PackageName' matched multiple apps: $($Apps.AppUserModelId)"
         } else {
             $Null = $Activator.ActivateApplication($Apps[0].AppUserModelId, $null, [Windows.ActivateOptions]::None, [ref]0)
         }
     }
     if($Protocol) {
         start $Protocol
     }
 }
`

