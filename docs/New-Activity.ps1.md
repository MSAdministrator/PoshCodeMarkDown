---
Author: adam driscoll
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3164
Published Date: 2012-01-15t16
Archived Date: 2015-11-13t03
---

# new-activity - 

## Description

wraps a pre-existing powershell cmdlet in a binary workflow activity and outputs a dll that can be used in visual studio.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 #
 #
 
 function global:Test-IsCommonParameter
 {
     param
     (   
         $Parameter
     )
     
     'WhatIf','Confirm','UseTransaction','Verbose','Debug','OutBuffer','OutVariable','WarningVariable','ErrorVariable','ErrorAction','WarningAction' -contains $Parameter.Name 
 }
 
 function global:New-Activity
 {
     param(
     [string]
     $CommandName,
     [String]
     $OutputAssembly
     )
     
     $Command = Get-Command -Name $CommandName
     
     $ParameterString = ""
     
     foreach($parameter in $Command.Parameters.Values )
     {
         if (Test-IsCommonParameter -Parameter $Parameter)
         {
             continue
         }
         $ParameterType = $parameter.ParameterType
         if ($ParameterType -match "switch")
         {
             $ParameterType = "SwitchParameter"
         }
         if ($ParameterType -match "scriptblock")
         {
             $ParameterType = "ScriptBlock"
         }
         
         $ParameterString += "
             [ParameterSpecificCategory]
             public InArgument<$ParameterType> $($Parameter.Name) {get;set;}
             
         "
     }
     
     $MethodString = ""
     
     foreach($parameter in $command.Parameters.Values)
     {
         if (Test-IsCommonParameter -Parameter $Parameter)
         {
             continue
         }
         $ParameterName = $parameter.Name
         $MethodString += "
             if (this.$ParameterName.Expression != null)
             {
                 shell2.AddParameter(`"$ParameterName`", this.$ParameterName.Get(context));
             }
         "
     }
     
     $ClassName = $Command.Name.Replace("-", "")
     $FullName = $command.ModuleName + "\" + $command.Name
     
     Add-Type -OutputType Library -OutputAssembly $OutputAssembly -ReferencedAssemblies "System.Activities","C:\windows\assembly\GAC_MSIL\Microsoft.PowerShell.Workflow.ServiceCore\3.0.0.0__31bf3856ad364e35\Microsoft.PowerShell.Workflow.ServiceCore.dll"  -TypeDefinition "
    
    using System.Activities;
    using System.Management.Automation;
    using Microsoft.PowerShell.Activities;
    using System;
    namespace CustomActitivies
    {
     public sealed class $ClassName : PSRemotingActivity
     {
         public $ClassName()
         {
             base.DisplayName = `"$ClassName`";
         }
     
     $ParameterString
         
         protected override ActivityImplementationContext GetPowerShell(NativeActivityContext context)
         {
             PowerShell shell = PowerShell.Create();
             PowerShell shell2 = shell.AddCommand(this.PSCommandName);
             $MethodString
             ActivityImplementationContext context2 = new ActivityImplementationContext();
             context2.PowerShellInstance = shell;
             return context2;
         }
         
         public override string PSCommandName
         {
             get
             {
                 return @`"$FullName`";
             }
         }
     }
     }
     "
 }
`

