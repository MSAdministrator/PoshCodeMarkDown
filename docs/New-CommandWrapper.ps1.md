---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6523
Published Date: 2016-09-20t22
Archived Date: 2016-09-26t22
---

# new-commandwrapper.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ##############################################################################
 ##
 ##
 ##
 ##############################################################################
 
 <#
 
 .SYNOPSIS
 
 Adds parameters and functionality to existing cmdlets and functions.
 
 .EXAMPLE
 
 New-CommandWrapper Get-Process `
       -AddParameter @{
           SortBy = {
               $newPipeline = {
                   __ORIGINAL_COMMAND__ | Sort-Object -Property $SortBy
               }
           }
       }
 
 This example adds a 'SortBy' parameter to Get-Process. It accomplishes
 this by adding a Sort-Object command to the pipeline.
 
 .EXAMPLE
 
 $parameterAttributes = @'
           [Parameter(Mandatory = $true)]
           [ValidateRange(50,75)]
           [Int]
 '@
 
 New-CommandWrapper Clear-Host `
       -AddParameter @{
           @{
               Name = 'MyMandatoryInt';
               Attributes = $parameterAttributes
           } = {
               Write-Host $MyMandatoryInt
               Read-Host "Press ENTER"
          }
       }
 
 This example adds a new mandatory 'MyMandatoryInt' parameter to
 Clear-Host. This parameter is also validated to fall within the range
 of 50 to 75. It doesn't alter the pipeline, but does display some
 information on the screen before processing the original pipeline.
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     $Name,
 
     [ScriptBlock] $Begin,
 
     [ScriptBlock] $Process,
 
     [ScriptBlock] $End,
 
     ##
     ##
     ##
     [HashTable] $AddParameter
 )
 
 Set-StrictMode -Version Latest
 
 $target = $Name
 $commandType = "Cmdlet"
 
 if(Test-Path function:\$Name)
 {
     $target = "$Name" + "-" + [Guid]::NewGuid().ToString().Replace("-","")
     Rename-Item function:\GLOBAL:$Name GLOBAL:$target
     $commandType = "Function"
 }
 
 $proxy = @'
 
 __CMDLET_BINDING_ATTRIBUTE__
 param(
 __PARAMETERS__
 )
 begin
 {
     try {
         __CUSTOM_BEGIN__
 
         $foreachObject = $executionContext.InvokeCommand.GetCmdlet(
             "Microsoft.PowerShell.Core\Foreach-Object")
 
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand(
             '__COMMAND_NAME__',
             [System.Management.Automation.CommandTypes]::__COMMAND_TYPE__)
 
         $targetParameters = @{}
         $PSBoundParameters.GetEnumerator() |
             & $foreachObject {
                 if($command.Parameters.ContainsKey($_.Key))
                 {
                     $targetParameters.Add($_.Key, $_.Value)
                 }
             }
 
         $newPipeline = { & $wrappedCmd @targetParameters }
         $finalPipeline = $newPipeline.ToString()
 
         __CUSTOM_PARAMETER_PROCESSING__
 
         $steppablePipeline = [ScriptBlock]::Create(
             $finalPipeline).GetSteppablePipeline()
         $steppablePipeline.Begin($PSCmdlet)
     } catch {
         throw
     }
 }
 
 process
 {
     try {
         __CUSTOM_PROCESS__
         $steppablePipeline.Process($_)
     } catch {
         throw
     }
 }
 
 end
 {
     try {
         __CUSTOM_END__
         $steppablePipeline.End()
     } catch {
         throw
     }
 }
 
 dynamicparam
 {
     $getCommand = $executionContext.InvokeCommand.GetCmdlet(
         "Microsoft.PowerShell.Core\Get-Command")
     $foreachObject = $executionContext.InvokeCommand.GetCmdlet(
         "Microsoft.PowerShell.Core\Foreach-Object")
     $whereObject = $executionContext.InvokeCommand.GetCmdlet(
         "Microsoft.PowerShell.Core\Where-Object")
 
     $command = & $getCommand __COMMAND_NAME__ -Type __COMMAND_TYPE__
     $targetParameters = @{}
     $PSBoundParameters.GetEnumerator() |
         & $foreachObject {
             if($command.Parameters.ContainsKey($_.Key))
             {
                 $targetParameters.Add($_.Key, $_.Value)
             }
         }
 
     $argList = @($targetParameters.GetEnumerator() |
         Foreach-Object {
             if($_.Value -isnot [switch] -or $_.Value)
             {
                 "-$($_.Key)"
             }
             if($_.Value -isnot [switch])
             {
                 $_.Value
             }
         })
 
     $command = $null
     try
     {
         $command = & $getCommand __COMMAND_NAME__ -Type __COMMAND_TYPE__ `
             -ArgumentList $argList
     }
     catch
     {
 
     }
 
     $dynamicParams = @($command.Parameters.GetEnumerator() |
         & $whereObject { $_.Value.IsDynamic })
 
     if ($dynamicParams.Length -gt 0)
     {
         $paramDictionary = `
             New-Object Management.Automation.RuntimeDefinedParameterDictionary
         foreach ($param in $dynamicParams)
         {
             $param = $param.Value
             $arguments = $param.Name, $param.ParameterType, $param.Attributes
             $newParameter = `
                 New-Object Management.Automation.RuntimeDefinedParameter `
                 $arguments
             $paramDictionary.Add($param.Name, $newParameter)
         }
         return $paramDictionary
     }
 }
 
 <#
 
 .ForwardHelpTargetName __COMMAND_NAME__
 .ForwardHelpCategory __COMMAND_TYPE__
 
 #>
 
 '@
 
 $originalCommand = Get-Command $target
 $metaData = New-Object System.Management.Automation.CommandMetaData `
     $originalCommand
 $proxyCommandType = [System.Management.Automation.ProxyCommand]
 
 $proxy = $proxy.Replace("__CMDLET_BINDING_ATTRIBUTE__",
     $proxyCommandType::GetCmdletBindingAttribute($metaData))
 $proxy = $proxy.Replace("__COMMAND_NAME__", $target)
 $proxy = $proxy.Replace("__COMMAND_TYPE__", $commandType)
 
 $newParamBlockCode = ""
 
 $beginAdditions = ""
 
 $currentParameter = $originalCommand.Parameters.Count
 if($AddParameter)
 {
     foreach($parameter in $AddParameter.Keys)
     {
         $parameterCode = $AddParameter[$parameter]
 
         if($parameter -is [Hashtable])
         {
             if($currentParameter -gt 0)
             {
                 $newParamBlockCode += ","
             }
 
             $newParamBlockCode += "`n`n    " +
                 $parameter.Attributes + "`n" +
                 '    $' + $parameter.Name
 
             $parameter = $parameter.Name
         }
         else
         {
             $newParameter =
                 New-Object System.Management.Automation.ParameterMetadata `
                     $parameter
             $metaData.Parameters.Add($parameter, $newParameter)
         }
 
         $parameterCode = $parameterCode.ToString()
 
         $templateCode = @"
 
         if(`$PSBoundParameters['$parameter'])
         {
             $parameterCode
 
             `$alteredPipeline = `$newPipeline.ToString()
             `$finalPipeline = `$alteredPipeline.Replace(
                 '__ORIGINAL_COMMAND__', `$finalPipeline)
         }
 "@
 
         $beginAdditions += $templateCode
         $currentParameter++
     }
 }
 
 $parameters = $proxyCommandType::GetParamBlock($metaData)
 if($newParamBlockCode) { $parameters += $newParamBlockCode }
 $proxy = $proxy.Replace('__PARAMETERS__', $parameters)
 
 $proxy = $proxy.Replace('__CUSTOM_BEGIN__', $Begin)
 $proxy = $proxy.Replace('__CUSTOM_PARAMETER_PROCESSING__', $beginAdditions)
 $proxy = $proxy.Replace('__CUSTOM_PROCESS__', $Process)
 $proxy = $proxy.Replace('__CUSTOM_END__', $End)
 
 Write-Verbose $proxy
 Set-Content function:\GLOBAL:$NAME $proxy
 
 if($commandType -eq "Cmdlet")
 {
     $originalCommand.Visibility = "Private"
 }
`

