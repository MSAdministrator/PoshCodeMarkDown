---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1885
Published Date: 
Archived Date: 2010-06-10t04
---

# sqlps2 - 

## Description

a c# source file created by running make-shell to create a sql server minishell

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 using System;
 using System.IO;
 using System.Collections;
 using System.Collections.Generic;
 using System.Collections.ObjectModel;
 using System.Resources;
 using System.Globalization;
 using System.Management.Automation;
 using System.Management.Automation.Runspaces;
 using System.Reflection;
 
 [assembly:RunspaceConfigurationType("Community.CustomRunspaceConfiguration")]
 
 namespace Community
 {
     /// <summary>
     /// CustomRunspaceConfiguration implements runspace configuration for current
     /// minishell.
     /// </summary>
     public class CustomRunspaceConfiguration : RunspaceConfiguration
     {
         /// <summary>
         /// Constructor is made private intentionally so that each instance of 
         /// CustomRunspaceConfiguration will be created through Create() function
         /// </summary>
         private CustomRunspaceConfiguration()
         {
         }
 
         /// <summary>
         /// This is the static function to create an instance of this type.
         /// </summary>
         /// <returns>runspace configuration instance created</returns>
         public new static RunspaceConfiguration Create()
         {
             CustomRunspaceConfiguration runspaceConfig = new CustomRunspaceConfiguration();
 
             return runspaceConfig;
         }
 
         private const string _shellId = "Community.sqlps2";
 
         /// <summary>
         /// Shell Id
         /// </summary>
         /// <value>Shell Id</value>
         public override string ShellId
         {
             get
             {
                 return _shellId;
             }
         }
 
         
         private CmdletConfigurationEntry[] _cmdletConfigurationEntries = new CmdletConfigurationEntry[] { new CmdletConfigurationEntry("Invoke-Sqlcmd", typeof(Microsoft.SqlServer.Management.PowerShell.GetScriptCommand), "Microsoft.SqlServer.Management.PSSnapins.dll-Help.xml"), 
             new CmdletConfigurationEntry("Invoke-PolicyEvaluation", typeof(Microsoft.SqlServer.Management.PowerShell.InvokePolicyEvaluationCommand), "Microsoft.SqlServer.Management.PSSnapins.dll-Help.xml"), 
             new CmdletConfigurationEntry("Encode-SqlName", typeof(Microsoft.SqlServer.Management.PowerShell.EncodeSqlName), "Microsoft.SqlServer.Management.PSProvider.dll-Help.xml"), 
             new CmdletConfigurationEntry("Decode-SqlName", typeof(Microsoft.SqlServer.Management.PowerShell.DecodeSqlName), "Microsoft.SqlServer.Management.PSProvider.dll-Help.xml"), 
             new CmdletConfigurationEntry("Convert-UrnToPath", typeof(Microsoft.SqlServer.Management.PowerShell.ConvertUrnToPath), "Microsoft.SqlServer.Management.PSProvider.dll-Help.xml"), 
             new CmdletConfigurationEntry("Start-Transcript", typeof(Microsoft.PowerShell.Commands.StartTranscriptCommand), "Microsoft.PowerShell.ConsoleHost.dll-Help.xml"), 
             new CmdletConfigurationEntry("Stop-Transcript", typeof(Microsoft.PowerShell.Commands.StopTranscriptCommand), "Microsoft.PowerShell.ConsoleHost.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Command", typeof(Microsoft.PowerShell.Commands.GetCommandCommand), "System.Management.Automation.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Help", typeof(Microsoft.PowerShell.Commands.GetHelpCommand), "System.Management.Automation.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-History", typeof(Microsoft.PowerShell.Commands.GetHistoryCommand), "System.Management.Automation.dll-Help.xml"), 
             new CmdletConfigurationEntry("Invoke-History", typeof(Microsoft.PowerShell.Commands.InvokeHistoryCommand), "System.Management.Automation.dll-Help.xml"), 
             new CmdletConfigurationEntry("Add-History", typeof(Microsoft.PowerShell.Commands.AddHistoryCommand), "System.Management.Automation.dll-Help.xml"), 
             new CmdletConfigurationEntry("ForEach-Object", typeof(Microsoft.PowerShell.Commands.ForEachObjectCommand), "System.Management.Automation.dll-Help.xml"), 
             new CmdletConfigurationEntry("Where-Object", typeof(Microsoft.PowerShell.Commands.WhereObjectCommand), "System.Management.Automation.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-PSDebug", typeof(Microsoft.PowerShell.Commands.SetPSDebugCommand), "System.Management.Automation.dll-Help.xml"), 
             new CmdletConfigurationEntry("Add-Content", typeof(Microsoft.PowerShell.Commands.AddContentCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Clear-Content", typeof(Microsoft.PowerShell.Commands.ClearContentCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Clear-ItemProperty", typeof(Microsoft.PowerShell.Commands.ClearItemPropertyCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Join-Path", typeof(Microsoft.PowerShell.Commands.JoinPathCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Convert-Path", typeof(Microsoft.PowerShell.Commands.ConvertPathCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Copy-ItemProperty", typeof(Microsoft.PowerShell.Commands.CopyItemPropertyCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-EventLog", typeof(Microsoft.PowerShell.Commands.GetEventLogCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-ChildItem", typeof(Microsoft.PowerShell.Commands.GetChildItemCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Content", typeof(Microsoft.PowerShell.Commands.GetContentCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-ItemProperty", typeof(Microsoft.PowerShell.Commands.GetItemPropertyCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-WmiObject", typeof(Microsoft.PowerShell.Commands.GetWmiObjectCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Move-ItemProperty", typeof(Microsoft.PowerShell.Commands.MoveItemPropertyCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Location", typeof(Microsoft.PowerShell.Commands.GetLocationCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-Location", typeof(Microsoft.PowerShell.Commands.SetLocationCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Push-Location", typeof(Microsoft.PowerShell.Commands.PushLocationCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Pop-Location", typeof(Microsoft.PowerShell.Commands.PopLocationCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("New-PSDrive", typeof(Microsoft.PowerShell.Commands.NewPSDriveCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Remove-PSDrive", typeof(Microsoft.PowerShell.Commands.RemovePSDriveCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-PSDrive", typeof(Microsoft.PowerShell.Commands.GetPSDriveCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Item", typeof(Microsoft.PowerShell.Commands.GetItemCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("New-Item", typeof(Microsoft.PowerShell.Commands.NewItemCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-Item", typeof(Microsoft.PowerShell.Commands.SetItemCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Remove-Item", typeof(Microsoft.PowerShell.Commands.RemoveItemCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Move-Item", typeof(Microsoft.PowerShell.Commands.MoveItemCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Rename-Item", typeof(Microsoft.PowerShell.Commands.RenameItemCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Copy-Item", typeof(Microsoft.PowerShell.Commands.CopyItemCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Clear-Item", typeof(Microsoft.PowerShell.Commands.ClearItemCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Invoke-Item", typeof(Microsoft.PowerShell.Commands.InvokeItemCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-PSProvider", typeof(Microsoft.PowerShell.Commands.GetPSProviderCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("New-ItemProperty", typeof(Microsoft.PowerShell.Commands.NewItemPropertyCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Split-Path", typeof(Microsoft.PowerShell.Commands.SplitPathCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Test-Path", typeof(Microsoft.PowerShell.Commands.TestPathCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Process", typeof(Microsoft.PowerShell.Commands.GetProcessCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Stop-Process", typeof(Microsoft.PowerShell.Commands.StopProcessCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Remove-ItemProperty", typeof(Microsoft.PowerShell.Commands.RemoveItemPropertyCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Rename-ItemProperty", typeof(Microsoft.PowerShell.Commands.RenameItemPropertyCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Resolve-Path", typeof(Microsoft.PowerShell.Commands.ResolvePathCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Service", typeof(Microsoft.PowerShell.Commands.GetServiceCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Stop-Service", typeof(Microsoft.PowerShell.Commands.StopServiceCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Start-Service", typeof(Microsoft.PowerShell.Commands.StartServiceCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Suspend-Service", typeof(Microsoft.PowerShell.Commands.SuspendServiceCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Resume-Service", typeof(Microsoft.PowerShell.Commands.ResumeServiceCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Restart-Service", typeof(Microsoft.PowerShell.Commands.RestartServiceCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-Service", typeof(Microsoft.PowerShell.Commands.SetServiceCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("New-Service", typeof(Microsoft.PowerShell.Commands.NewServiceCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-Content", typeof(Microsoft.PowerShell.Commands.SetContentCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-ItemProperty", typeof(Microsoft.PowerShell.Commands.SetItemPropertyCommand), "Microsoft.PowerShell.Commands.Management.dll-Help.xml"), 
             new CmdletConfigurationEntry("Format-Default", typeof(Microsoft.PowerShell.Commands.FormatDefaultCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Format-List", typeof(Microsoft.PowerShell.Commands.FormatListCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Format-Custom", typeof(Microsoft.PowerShell.Commands.FormatCustomCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Format-Table", typeof(Microsoft.PowerShell.Commands.FormatTableCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Format-Wide", typeof(Microsoft.PowerShell.Commands.FormatWideCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Out-Null", typeof(Microsoft.PowerShell.Commands.OutNullCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Out-Default", typeof(Microsoft.PowerShell.Commands.OutDefaultCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Out-Host", typeof(Microsoft.PowerShell.Commands.OutHostCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Out-File", typeof(Microsoft.PowerShell.Commands.OutFileCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Out-Printer", typeof(Microsoft.PowerShell.Commands.OutPrinterCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Out-String", typeof(Microsoft.PowerShell.Commands.OutStringCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Out-LineOutput", typeof(Microsoft.PowerShell.Commands.OutLineOutputCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Add-Member", typeof(Microsoft.PowerShell.Commands.AddMemberCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Compare-Object", typeof(Microsoft.PowerShell.Commands.CompareObjectCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("ConvertTo-Html", typeof(Microsoft.PowerShell.Commands.ConvertToHtmlCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Export-Csv", typeof(Microsoft.PowerShell.Commands.ExportCsvCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Import-Csv", typeof(Microsoft.PowerShell.Commands.ImportCsvCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Export-Alias", typeof(Microsoft.PowerShell.Commands.ExportAliasCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Invoke-Expression", typeof(Microsoft.PowerShell.Commands.InvokeExpressionCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Alias", typeof(Microsoft.PowerShell.Commands.GetAliasCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Culture", typeof(Microsoft.PowerShell.Commands.GetCultureCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Date", typeof(Microsoft.PowerShell.Commands.GetDateCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Host", typeof(Microsoft.PowerShell.Commands.GetHostCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Member", typeof(Microsoft.PowerShell.Commands.GetMemberCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-UICulture", typeof(Microsoft.PowerShell.Commands.GetUICultureCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Unique", typeof(Microsoft.PowerShell.Commands.GetUniqueCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Import-Alias", typeof(Microsoft.PowerShell.Commands.ImportAliasCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Select-String", typeof(Microsoft.PowerShell.Commands.SelectStringCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Measure-Object", typeof(Microsoft.PowerShell.Commands.MeasureObjectCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("New-Alias", typeof(Microsoft.PowerShell.Commands.NewAliasCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("New-TimeSpan", typeof(Microsoft.PowerShell.Commands.NewTimeSpanCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Read-Host", typeof(Microsoft.PowerShell.Commands.ReadHostCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-Alias", typeof(Microsoft.PowerShell.Commands.SetAliasCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-Date", typeof(Microsoft.PowerShell.Commands.SetDateCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Start-Sleep", typeof(Microsoft.PowerShell.Commands.StartSleepCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Tee-Object", typeof(Microsoft.PowerShell.Commands.TeeObjectCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Measure-Command", typeof(Microsoft.PowerShell.Commands.MeasureCommandCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Update-TypeData", typeof(Microsoft.PowerShell.Commands.UpdateTypeDataCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Update-FormatData", typeof(Microsoft.PowerShell.Commands.UpdateFormatDataCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Write-Host", typeof(Microsoft.PowerShell.Commands.WriteHostCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Write-Progress", typeof(Microsoft.PowerShell.Commands.WriteProgressCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("New-Object", typeof(Microsoft.PowerShell.Commands.NewObjectCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Select-Object", typeof(Microsoft.PowerShell.Commands.SelectObjectCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Group-Object", typeof(Microsoft.PowerShell.Commands.GroupObjectCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Sort-Object", typeof(Microsoft.PowerShell.Commands.SortObjectCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Variable", typeof(Microsoft.PowerShell.Commands.GetVariableCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("New-Variable", typeof(Microsoft.PowerShell.Commands.NewVariableCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-Variable", typeof(Microsoft.PowerShell.Commands.SetVariableCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Remove-Variable", typeof(Microsoft.PowerShell.Commands.RemoveVariableCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Clear-Variable", typeof(Microsoft.PowerShell.Commands.ClearVariableCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Export-Clixml", typeof(Microsoft.PowerShell.Commands.ExportClixmlCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Import-Clixml", typeof(Microsoft.PowerShell.Commands.ImportClixmlCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Write-Debug", typeof(Microsoft.PowerShell.Commands.WriteDebugCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Write-Verbose", typeof(Microsoft.PowerShell.Commands.WriteVerboseCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Write-Warning", typeof(Microsoft.PowerShell.Commands.WriteWarningCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Write-Error", typeof(Microsoft.PowerShell.Commands.WriteErrorCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Write-Output", typeof(Microsoft.PowerShell.Commands.WriteOutputCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-TraceSource", typeof(Microsoft.PowerShell.Commands.GetTraceSourceCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-TraceSource", typeof(Microsoft.PowerShell.Commands.SetTraceSourceCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Trace-Command", typeof(Microsoft.PowerShell.Commands.TraceCommandCommand), "Microsoft.PowerShell.Commands.Utility.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Acl", typeof(Microsoft.PowerShell.Commands.GetAclCommand), "Microsoft.PowerShell.Security.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-Acl", typeof(Microsoft.PowerShell.Commands.SetAclCommand), "Microsoft.PowerShell.Security.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-PfxCertificate", typeof(Microsoft.PowerShell.Commands.GetPfxCertificateCommand), "Microsoft.PowerShell.Security.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-Credential", typeof(Microsoft.PowerShell.Commands.GetCredentialCommand), "Microsoft.PowerShell.Security.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-ExecutionPolicy", typeof(Microsoft.PowerShell.Commands.GetExecutionPolicyCommand), "Microsoft.PowerShell.Security.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-ExecutionPolicy", typeof(Microsoft.PowerShell.Commands.SetExecutionPolicyCommand), "Microsoft.PowerShell.Security.dll-Help.xml"), 
             new CmdletConfigurationEntry("Get-AuthenticodeSignature", typeof(Microsoft.PowerShell.Commands.GetAuthenticodeSignatureCommand), "Microsoft.PowerShell.Security.dll-Help.xml"), 
             new CmdletConfigurationEntry("Set-AuthenticodeSignature", typeof(Microsoft.PowerShell.Commands.SetAuthenticodeSignatureCommand), "Microsoft.PowerShell.Security.dll-Help.xml"), 
             new CmdletConfigurationEntry("ConvertFrom-SecureString", typeof(Microsoft.PowerShell.Commands.ConvertFromSecureStringCommand), "Microsoft.PowerShell.Security.dll-Help.xml"), 
             new CmdletConfigurationEntry("ConvertTo-SecureString", typeof(Microsoft.PowerShell.Commands.ConvertToSecureStringCommand), "Microsoft.PowerShell.Security.dll-Help.xml") };
 
         private RunspaceConfigurationEntryCollection<CmdletConfigurationEntry> _cmdlets;
 
         /// <summary>
         /// Cmdlets for this minishell
         /// </summary>
         /// <value>a collection of cmdlet configuration entries</value>
         public override RunspaceConfigurationEntryCollection<CmdletConfigurationEntry> Cmdlets
         {
             get
             {
                 if (_cmdlets == null)
                 {
                     _cmdlets = new RunspaceConfigurationEntryCollection<CmdletConfigurationEntry>(_cmdletConfigurationEntries);
                 }
 
                 return _cmdlets;
             }
         }
 
         private ProviderConfigurationEntry[] _providerConfigurationEntries = new ProviderConfigurationEntry[] { new ProviderConfigurationEntry("SqlServer", typeof(Microsoft.SqlServer.Management.PowerShell.SqlServerProvider), "Microsoft.SqlServer.Management.PSProvider.dll-Help.xml"), 
             new ProviderConfigurationEntry("Alias", typeof(Microsoft.PowerShell.Commands.AliasProvider), "System.Management.Automation.dll-Help.xml"), 
             new ProviderConfigurationEntry("Environment", typeof(Microsoft.PowerShell.Commands.EnvironmentProvider), "System.Management.Automation.dll-Help.xml"), 
             new ProviderConfigurationEntry("FileSystem", typeof(Microsoft.PowerShell.Commands.FileSystemProvider), "System.Management.Automation.dll-Help.xml"), 
             new ProviderConfigurationEntry("Function", typeof(Microsoft.PowerShell.Commands.FunctionProvider), "System.Management.Automation.dll-Help.xml"), 
             new ProviderConfigurationEntry("Registry", typeof(Microsoft.PowerShell.Commands.RegistryProvider), "System.Management.Automation.dll-Help.xml"), 
             new ProviderConfigurationEntry("Variable", typeof(Microsoft.PowerShell.Commands.VariableProvider), "System.Management.Automation.dll-Help.xml"), 
             new ProviderConfigurationEntry("Certificate", typeof(Microsoft.PowerShell.Commands.CertificateProvider), "Microsoft.PowerShell.Security.dll-Help.xml") };
 
         private RunspaceConfigurationEntryCollection<ProviderConfigurationEntry> _providers;
 
         /// <summary>
         /// Providers for this minishell
         /// </summary>
         /// <value>a collection of provider configuration entries</value>
         public override RunspaceConfigurationEntryCollection<ProviderConfigurationEntry> Providers
         {
             get
             {
                 if (_providers == null)
                 {
                     _providers = new RunspaceConfigurationEntryCollection<ProviderConfigurationEntry>(_providerConfigurationEntries);
                 }
 
                 return _providers;
             }
         }
 
         private AssemblyConfigurationEntry[] _assemblyConfigurationEntries = new AssemblyConfigurationEntry[] { new AssemblyConfigurationEntry("Microsoft.SqlServer.Management.PSSnapins, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.Management.PSSnapins.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.Management.PSProvider, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.Management.PSProvider.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.Smo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.Smo.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.Dmf, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.Dmf.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.SqlWmiManagement, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.SqlWmiManagement.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.ConnectionInfo, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.ConnectionInfo.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.SmoExtended, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.SmoExtended.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.Management.Sdk.Sfc, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.Management.Sdk.Sfc.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.SqlEnum, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.SqlEnum.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.RegSvrEnum, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.RegSvrEnum.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.WmiEnum, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.WmiEnum.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.ServiceBrokerEnum, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.ServiceBrokerEnum.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.ConnectionInfoExtended, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.ConnectionInfoExtended.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.Management.Collector, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.Management.Collector.dll"), 
             new AssemblyConfigurationEntry("Microsoft.SqlServer.Management.CollectorEnum, Version=10.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91", "Microsoft.SqlServer.Management.CollectorEnum.dll"), 
             new AssemblyConfigurationEntry("Microsoft.PowerShell.ConsoleHost, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35", "Microsoft.PowerShell.ConsoleHost.dll"), 
             new AssemblyConfigurationEntry("System.Management.Automation, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35", "System.Management.Automation.dll"), 
             new AssemblyConfigurationEntry("Microsoft.PowerShell.Commands.Management, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35", "Microsoft.PowerShell.Commands.Management.dll"), 
             new AssemblyConfigurationEntry("Microsoft.PowerShell.Commands.Utility, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35", "Microsoft.PowerShell.Commands.Utility.dll"), 
             new AssemblyConfigurationEntry("Microsoft.PowerShell.Security, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35", "Microsoft.PowerShell.Security.dll") };
 
         private RunspaceConfigurationEntryCollection<AssemblyConfigurationEntry> _assemblies;
 
         /// <summary>
         /// Type resolution assemblies for this minishell
         /// </summary>
         /// <value>a collection of assembly configuration entries</value>
         public override RunspaceConfigurationEntryCollection<AssemblyConfigurationEntry> Assemblies
         {
             get
             {
                 if (_assemblies == null)
                 {
                     _assemblies = new RunspaceConfigurationEntryCollection<AssemblyConfigurationEntry>(_assemblyConfigurationEntries);
                 }
 
                 return _assemblies;
             }
         }
 
 
         
         private TypeConfigurationEntry[] _typeConfigurationEntries = new TypeConfigurationEntry[] { new TypeConfigurationEntry(@"SQLProvider.Types.ps1xml"), new TypeConfigurationEntry(@"types.ps1xml") };
 
         private RunspaceConfigurationEntryCollection<TypeConfigurationEntry> _types;
 
         /// <summary>
         /// Types for this minishell
         /// </summary>
         /// <value>a collection of type configuration entries</value>
         public override RunspaceConfigurationEntryCollection<TypeConfigurationEntry> Types
         {
             get
             {
                 if (_types == null)
                 {
                     _types = new RunspaceConfigurationEntryCollection<TypeConfigurationEntry>(_typeConfigurationEntries);
                 }
 
                 return _types;
             }
         }
 
         private FormatConfigurationEntry[] _formatConfigurationEntries = new FormatConfigurationEntry[] { new FormatConfigurationEntry(@"SQLProvider.Format.ps1xml"), new FormatConfigurationEntry(@"DotNetTypes.format.ps1xml"), new FormatConfigurationEntry(@"PowerShellCore.format.ps1xml"), new FormatConfigurationEntry(@"PowerShellTrace.format.ps1xml"), new FormatConfigurationEntry(@"FileSystem.format.ps1xml"), new FormatConfigurationEntry(@"Registry.format.ps1xml"), new FormatConfigurationEntry(@"Certificate.format.ps1xml"), new FormatConfigurationEntry(@"Help.format.ps1xml") };
 
         private RunspaceConfigurationEntryCollection<FormatConfigurationEntry> _formats;
 
         /// <summary>
         /// Formats for this minishell
         /// </summary>
         /// <value>a collection of format configuration entries</value>
         public override RunspaceConfigurationEntryCollection<FormatConfigurationEntry> Formats
         {
             get
             {
                 if (_formats == null)
                 {
                     _formats = new RunspaceConfigurationEntryCollection<FormatConfigurationEntry>(_formatConfigurationEntries);
                 }
 
                 return _formats;
             }
         }
 
 
         
 
 
         private RunspaceConfigurationEntryCollection<ScriptConfigurationEntry> _scripts;
 
         /// <summary>
         /// Pre-defined scripts for this minishell
         /// </summary>
         /// <value>a collection of script configuration entries</value>
         public override RunspaceConfigurationEntryCollection<ScriptConfigurationEntry> Scripts
         {
             get
             {
                 if (_scripts == null)
                 {
                     _scripts = new RunspaceConfigurationEntryCollection<ScriptConfigurationEntry>();
 
                     string[] scriptNames = null;
                     string[] scripts = null;
 
                     GetResources(ScriptResourceName, ref scriptNames, ref scripts);
 
                     if (scripts != null)
                     {
                         for (int i = 0; i < scripts.Length; i++)
                         {
                             ScriptConfigurationEntry data = new ScriptConfigurationEntry(scriptNames[i], scripts[i]);
                             _scripts.Append(data);
                         }
                     }
                 }
 
                 return _scripts;
             }
         }
 
         private RunspaceConfigurationEntryCollection<ScriptConfigurationEntry> _initializationScripts;
 
         /// <summary>
         /// Initialization scripts to be run at minishell start up.
         /// </summary>
         /// <value>a collection of initialization scripts</value>
         public override RunspaceConfigurationEntryCollection<ScriptConfigurationEntry> InitializationScripts
         {
             get
             {
                 if (_initializationScripts == null)
                 {
                     _initializationScripts = new RunspaceConfigurationEntryCollection<ScriptConfigurationEntry>();
 
                     Dictionary<string, object> profile = GetResources(ProfileResourceName, CultureInfo.InvariantCulture);
 
                     if (profile == null)
                         return null;
 
                     if (profile != null && profile.ContainsKey(ProfileResourceName))
                     {
                         _initializationScripts.Append(new ScriptConfigurationEntry(ProfileResourceName, (string)profile[ProfileResourceName]));
                     }
                 }
 
                 return _initializationScripts;
             }
         }
 
 
 
         static private string _shellHelp = null;
         static private string ShellHelp
         {
             get
             {
                 if(_shellHelp == null)
                     LoadHelpResource();
 
                 return _shellHelp;
             }
         }
 
         static private string _shellBanner = null;
         static private string ShellBanner
         {
             get
             {
                 if(_shellBanner == null)
                     LoadHelpResource();
 
                 return _shellBanner;
             }
         }
 
         static private void LoadHelpResource()
         {
             Dictionary<string,object> helpResources = GetResources(HelpResourceName, CultureInfo.CurrentUICulture);
 
             if(helpResources != null)
             {
                 if(helpResources.ContainsKey(ShellHelpResourceKey))
                 {
                     _shellHelp = (string) helpResources[ShellHelpResourceKey];
                 }
 
                 if(helpResources.ContainsKey(ShellBannerResourceKey))
                 {
                     _shellBanner = (string) helpResources[ShellBannerResourceKey];
                 }
             }
 
             return;
         }
 
 
 
         private const string ScriptResourceName = "script";
         private const string ProfileResourceName = "initialization";
         private const string HelpResourceName = "help";
         private const string ShellHelpResourceKey = "ShellHelp";
         private const string ShellBannerResourceKey = "ShellBanner";
         private const string ResourceListKey = "__resourceList__";
 
         /// <summary>
         /// Return all resources in ordered name/value pairs. Order of resources are 
         /// defined in resourceListKey. This will look for resources for invariant
         /// culture.
         /// </summary>
         private static void GetResources(string resourceType, ref string[] names, ref string[] values)
         {
             names = null; 
             values = null;
 
             Dictionary<string, object> resources = GetResources(resourceType, CultureInfo.InvariantCulture);
 
             if(resources == null)
                 return;
 
             if(!resources.ContainsKey(ResourceListKey))
             {
                 return;
             }
 
             names = (string[])resources[ResourceListKey];
 
             if(names == null || names.Length == 0)
                 return;
 
             values = new string[names.Length];
 
             for (int i = 0; i < names.Length; i++)
             {
                 values[i] = (string)resources[names[i]];
             }
 
             return;
         }
 
         /// <summary>
         /// Return all resources in defined in current assembly in the format of a string/object dictionary.
         /// </summary>
         private static Dictionary<string, object> GetResources(string resourceType, CultureInfo culture)
         {
             ResourceManager resMgr = new ResourceManager(resourceType, Assembly.GetExecutingAssembly());
             if (resMgr == null)
                 return null;
 
             ResourceSet resSet = null;
             try
             {
                 resSet = resMgr.GetResourceSet(culture, true, true);
             }
             catch (MissingManifestResourceException)
             {
                 return null;
             }
 
             if (resSet == null)
                 return null;
 
             Dictionary<string, object> result = new Dictionary<string, object>(StringComparer.OrdinalIgnoreCase);
 
             IDictionaryEnumerator id = resSet.GetEnumerator();
             while (id.MoveNext())
             {
                 result[(string)id.Key] = id.Value;
             }
 
             return result;
         }
 
 
         /// <summary>
         /// This is the main function for minishell. It will start msh console shell with 
         /// a runspace configuration instance for current minishell.
         /// </summary>
         /// <param name="args">args to main</param>
         /// <returns>minishell execution status</returns>
         [LoaderOptimization(LoaderOptimization.MultiDomain)]
         public static int Main(string[] args)
         {
             RunspaceConfiguration configuration = Create();
 
             return Microsoft.PowerShell.ConsoleShell.Start(configuration, ShellBanner, ShellHelp, args);
         }
     }
 }
`

