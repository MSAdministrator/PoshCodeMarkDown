---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 3290
Published Date: 2012-03-15t12
Archived Date: 2012-03-28t09
---

# convertto-iseaddon - 

## Description

temporary listing for an update to convertto-iseaddon for showui 1.3 (for powershell ise 3)

## Comments



## Usage



## TODO



## function

`convertto-iseaddon`

## Code

`#
 #
 function ConvertTo-ISEAddOn
 {
 #.Synopsis
 #.Example
 #.Example
 
    [CmdletBinding(DefaultParameterSetName="CreateOnly")]
    param(
       [Parameter(Mandatory=$true, ParameterSetName="DisplayNow", Position = 0)]
       [string]$DisplayName,
       
       [Parameter(Mandatory=$true, Position = 1)]
       [ScriptBlock]$ScriptBlock,
 
       [Parameter(ParameterSetName="DisplayNow")]
       [switch]$Vertical,
 
       [Parameter(ParameterSetName="DisplayNow")]
       [switch]$Hidden
    )
 
    begin {
       if ($psVersionTable.PSVersion -lt "3.0") {
          Write-Error "Ise Window Add ons were not added until version 3.0."
          return
       }
    }
 
    process {
       $addOnNumber = Get-Random
       $addOnCode =@"
 namespace ShowISEAddOns
 {
    using System;
    using System.Collections.ObjectModel;
    using System.ComponentModel;
    using System.Management.Automation;
    using System.Management.Automation.Runspaces;
    using System.Windows;
    using System.Windows.Controls;
    using System.Windows.Data;
    using Microsoft.PowerShell.Host.ISE;
    using System.Collections.Generic;
    using System.Windows.Input;
    using System.Text;
 
    public class ShowUIIseAddOn${addOnNumber} : UserControl, IAddOnToolHostObject
    {
       ObjectModelRoot hostObject;
       public ObjectModelRoot HostObject
       {
          get
          {
             return this.hostObject;
          }
          set
          {
             this.hostObject = value;
             this.hostObject.CurrentPowerShellTab.PropertyChanged += new PropertyChangedEventHandler(CurrentPowerShellTab_PropertyChanged);
          }
       }
 
       private void CurrentPowerShellTab_PropertyChanged(object sender, PropertyChangedEventArgs e)
       {
          if (e.PropertyName == "CanInvoke" && this.hostObject.CurrentPowerShellTab.CanInvoke)
          {
             if (this.Content != null && this.Content is UIElement )
             {                     
                (this.Content as UIElement).IsEnabled = true; 
             }
          } else {
             if (this.Content != null && this.Content is UIElement)
             { 
                (this.Content as UIElement).IsEnabled = false; 
             }
          }
       }
 
 //      protected override void OnInitialized(EventArgs e) 
       public ShowUIIseAddOn${addOnNumber}()
       {
          if (Runspace.DefaultRunspace == null ||
             Runspace.DefaultRunspace.ApartmentState != System.Threading.ApartmentState.STA ||
             Runspace.DefaultRunspace.ThreadOptions != PSThreadOptions.UseCurrentThread)
          {
             var iss = InitialSessionState.CreateDefault();
             iss.ImportPSModule(new string[] { @"$(Split-Path (Get-Module ShowUI).Path)" });
             iss.Variables.Add( new SessionStateVariableEntry( "PSISE", HostObject, "", ScopedItemOptions.Constant ) );
             Runspace rs  = RunspaceFactory.CreateRunspace(iss);
             rs.ApartmentState = System.Threading.ApartmentState.STA;
             rs.ThreadOptions = PSThreadOptions.UseCurrentThread;
             rs.Open();
             Runspace.DefaultRunspace = rs;
          }
          
          PowerShell psCmd = PowerShell.Create().AddScript(@"
 $($ScriptBlock.ToString().Replace('"','""'))
 ");
          psCmd.Runspace = Runspace.DefaultRunspace;
          try 
          { 
             this.Content = psCmd.Invoke<UIElement>()[0];
          } 
          catch(Exception ex)
          {
             HostObject.CurrentPowerShellTab.Invoke("Write-Error '" + ex.Message + "\n\n" + ex.StackTrace + "'" );
          } 
       }        
    }
 }
 "@
 
       $presentationFramework = [System.Windows.Window].Assembly.FullName
       $presentationCore = [System.Windows.UIElement].Assembly.FullName
       $windowsBase=[System.Windows.DependencyObject].Assembly.FullName
       $gPowerShell=[Microsoft.PowerShell.Host.ISE.PowerShellTab].Assembly.FullName
       $systemXaml=[system.xaml.xamlreader].Assembly.FullName
       $systemManagementAutomation=[psobject].Assembly.FullName
       
       $CodeFile = Join-Path ([System.IO.Path]::GetTempPath()) ([System.IO.Path]::GetRandomFileName() + ".cs")
       Set-Content $CodeFile -Value $AddOnCode
       Write-Verbose $CodeFile
       $AddOnType = Add-Type -Path $CodeFile -PassThru -ReferencedAssemblies $systemManagementAutomation,$presentationFramework,$presentationCore,$windowsBase,$gPowerShell,$systemXaml -IgnoreWarnings
        
       if($PSCmdlet.ParameterSetName -eq "DisplayNow") { 
          if ($Vertical) {
             $psISE.CurrentPowerShellTab.VerticalAddOnTools.Add($displayName, $AddOnType, !$Hidden)
          } else {
             $psISE.CurrentPowerShellTab.HorizontalAddOnTools.Add($displayName, $AddOnType, !$Hidden)
          } 
       } else {
          $AddOnType
       }  
    }
 }
`

