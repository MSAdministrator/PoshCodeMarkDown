---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1288
Published Date: 
Archived Date: 2009-10-17t19
---

# remove-aeropeek - 

## Description

exclude a window from aero peek so it stays visible when you press win+space in windows 7 (of course, there’s lots of other possibilities here, but i’ll let others explore them).

## Comments



## Usage



## TODO



## function

`remove-aeropeek`

## Code

`#
 #
 function Remove-AeroPeek {
 PARAM(
    [string]$Title="*", 
    [string]$Process="*"
 )
 BEGIN {
 try { 
    $null = [Huddled.DWM]
 } catch { 
 Add-Type -Type @"
 using System;
 using System.Runtime.InteropServices;
 
 namespace Huddled {
    public static class Dwm {
       [DllImport("dwmapi.dll", PreserveSig = false)]
       public static extern int DwmSetWindowAttribute(IntPtr hwnd, int attr, ref int attrValue, int attrSize);
 
       [Flags]
       public enum DwmWindowAttribute
       {
          NCRenderingEnabled = 1,
          NCRenderingPolicy,
          TransitionsForceDisabled,
          AllowNCPaint,
          CaptionButtonBounds,
          NonClientRtlLayout,
          ForceIconicRepresentation,
          Flip3DPolicy,
          ExtendedFrameBounds,
          HasIconicBitmap,
          DisallowPeek,
          ExcludedFromPeek,
          Last
       }
 
       [Flags]
       public enum DwmNCRenderingPolicy
       {
          UseWindowStyle,
          Disabled,
          Enabled,
          Last
       }
 
       public static void RemoveFromAeroPeek(IntPtr Hwnd) //Hwnd is the handle to your window
       {
          int renderPolicy = (int)DwmNCRenderingPolicy.Enabled;
          DwmSetWindowAttribute(Hwnd, (int)DwmWindowAttribute.ExcludedFromPeek, ref renderPolicy, sizeof(int));
       }
    }
 }
 "@
 
 [Reflection.Assembly]::Load("UIAutomationClient, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35")
 function global:Select-Window {
 PARAM( [string]$Title="*", 
        [string]$Process="*", 
        [switch]$Recurse,
        [System.Windows.Automation.AutomationElement]$Parent = [System.Windows.Automation.AutomationElement]::RootElement ) 
 PROCESS {
    $search = "Children"
    if($Recurse) { $search = "Descendants" }
    
    $Parent.FindAll( $search, [System.Windows.Automation.Condition]::TrueCondition ) | 
    Add-Member -Type ScriptProperty -Name Title     -Value {
                $this.GetCurrentPropertyValue([System.Windows.Automation.AutomationElement]::NameProperty)} -Passthru |
    Add-Member -Type ScriptProperty -Name Handle    -Value {
                $this.GetCurrentPropertyValue([System.Windows.Automation.AutomationElement]::NativeWindowHandleProperty)} -Passthru |
    Add-Member -Type ScriptProperty -Name ProcessId -Value {
                $this.GetCurrentPropertyValue([System.Windows.Automation.AutomationElement]::ProcessIdProperty)} -Passthru |
 
    Where-Object {
       ((Get-Process -Id $_.ProcessId).ProcessName -like $Process) -and ($_.Title -like $Title)
    }
 }}
 }
 }
 PROCESS {
    Foreach($window in Select-Window -Process $Process -Title $Title) { 
       [Huddled.Dwm]::RemoveFromAeroPeek( $window.Handle ) 
    }
 }
 }
 
`

