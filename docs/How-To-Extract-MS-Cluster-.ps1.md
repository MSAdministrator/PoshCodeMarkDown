---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4135
Published Date: 
Archived Date: 2017-03-25t18
---

#  - buildpowershellclustercommands.ps1

## Description

how-to extract ms cluster resources and create powershell commands out of them.

## Comments

this script runs a series of 'get' cmdlet's to inventory a ms cluster and turns around

## Usage



## TODO



## script

``

## Code

`#
 #
 #######################################################################################################
 #
 #
 #
 #
 #
 #######################################################################################################
 Clear-Host
 [BOOL]  $RightNode = $false
 $date4file = Get-Date -uformat "%Y-%m-%d-%H-%M-%S"
 $now = Get-Date
 $cmdfilepath = 'c:\<CHANGEME!>\logs\cluster\'
 $synopsis = @"
 
 The purpose of this file is to provide you a way to create cluster resources that existed in the cluster
 named in the front of this file name.
 
 Commands below need to be run inside Powershell console or as a group inside a PS1 file.
 Start Powershell by navigating to [Start > All Programs > Accessories > Windows Powershell], right click
 on Powershell and select [Run as Administrator].
 
 Set execution policy by running following command, [set-executionpolicy unrestricted]. < no brackets
 Import cluster support by running [import-module failoverclusters]. < no brackets
 
 All resources will be in an offline state when they are done being created.  You will need to open the Failover Clustering
 GUI and bring them online.
 
 Windows 2008 R2 is the minumum OS required to run the cluster Powershell CMDLETs.
 Microsoft recommends creating disk resources using the failover cluster management GUI.
 
 Make sure to create any resources that other resources are dependent on first.
 This script does NOT ensure that the order of Powershell command executions is in order of their respective dependency chains.
 
 Notes:
 
 Some resource parameters cannot be set but this script does not filter those.  You may see errors when you try to set a parameter
 that is read-only or the cast is not valid.  Some errors you may see are:
 
 Specified cast is not valid.
 The value of read-only property '<property name>' cannot be changed.
 
 These errors are for the most part expected but all should be investigated.
 #################################################################################################################################
 "@
 
 Import-Module FailOverClusters
 
 $ClusterGroups = Get-ClusterGroup
 ForEach ($ClusterGroup in $ClusterGroups) {
     If (($env:COMPUTERNAME -eq $ClusterGroup.OwnerNode) -and ($ClusterGroup.Name -eq 'Cluster Group')){
         $RightNode = $true
     }
 }
 If ($RightNode){
     $clustervip = (Get-Cluster).name
     $cmdfile = "$clustervip-powershell-resource-cmds.txt"
 
     del $cmdfilepath\$cmdfile -ErrorAction SilentlyContinue
 
     Out-File "$cmdfilepath\$cmdfile" -Encoding ASCII
     Add-Content $cmdfilepath\$cmdfile "This file was created/updated on $now" -Encoding ASCII
     Add-Content $cmdfilepath\$cmdfile $synopsis -Encoding ASCII
 
     ForEach ($ClusterGroup in $ClusterGroups) {
         $String = 'Add-ClusterGroup -Name ' + [char]34 + $ClusterGroup.name + [char]34
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
     }
     Add-Content $cmdfilepath\$cmdfile "#" -Encoding ASCII
     $ResourceTypes = Get-ClusterResourceType
     ForEach ($ResourceType in $ResourceTypes) {
         $String = 'Add-ClusterResourceType -Name ' + [char]34 + $ResourceType.name + [char]34 + ' -Dll ' + $ResourceType.DllName
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
     }
     Add-Content $cmdfilepath\$cmdfile "#" -Encoding ASCII
     $ClusterResources = Get-ClusterResource
     ForEach ($ClusterResource in $ClusterResources) {
         $String = 'Add-ClusterResource -Name ' + [char]34 + $ClusterResource.name + [char]34 + ' -Group ' + [char]34 + $ClusterResource.ownergroup + [char]34 + ' -ResourceType ' + [char]34 + $ClusterResource.resourcetype + [char]34
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.DeadlockTimeout=' + $ClusterResource.DeadlockTimeout + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.IsAlivePollInterval=' + $ClusterResource.IsAlivePollInterval + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.LooksAlivePollInterval=' + $ClusterResource.LooksAlivePollInterval + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.PendingTimeout=' + $ClusterResource.PendingTimeout + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.PersistentState=' + $ClusterResource.PersistentState + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.RestartAction=' + $ClusterResource.RestartAction + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.RestartDelay=' + $ClusterResource.RestartDelay + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.RestartPeriod=' + $ClusterResource.RestartPeriod + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.RestartThreshold=' + $ClusterResource.RestartThreshold + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.RetryPeriodOnFailure=' + $ClusterResource.RetryPeriodOnFailure + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $String = 'Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34 + ' | % {$_.SeparateMonitor=$' + $ClusterResource.SeparateMonitor + '}'
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         Add-Content $cmdfilepath\$cmdfile "#" -Encoding ASCII
         $String = '$ResourceParameters = Get-ClusterResource ' + [char]34 + $ClusterResource.Name + [char]34
         Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         $parmcounter = 1
         $ResourceParameters = Get-ClusterResource $ClusterResource.Name | Get-ClusterParameter
         ForEach ($ResourceParameter in $ResourceParameters) {
             If (($ClusterResource.ResourceType -like 'IP Address') -and (($ResourceParameter.Name -like 'Address') -or ($ResourceParameter.Name -like 'SubnetMask') -or ($ResourceParameter.Name -like 'Network'))){
                 $String = '$parm' + $parmcounter + ' = New-Object Microsoft.FailoverClusters.PowerShell.ClusterParameter -ArgumentList $ResourceParameters,' + [char]34 + $ResourceParameter.Name + [char]34 + ',' + [char]34 + $ResourceParameter.value + [char]34
                 Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
                 $parmcounter ++
                 If ($parmcounter -gt 3){
                     $String = '$parms = $parm1,$parm2,$parm3'
                     Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
                     $String = '$parms | Set-ClusterParameter'
                     Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
                     Add-Content $cmdfilepath\$cmdfile "#" -Encoding ASCII
                 }
             }
             Else{
                 $String = 'Set-ClusterParameter -InputObject $ResourceParameters' + ' -Name ' + [char]34 + $ResourceParameter.name + [char]34 + ' -Value ' + [char]34 + $ResourceParameter.value + [char]34
                 Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
             }
         }
         Add-Content $cmdfilepath\$cmdfile "#" -Encoding ASCII
     }
     $ClusterResourceDependencies = Get-ClusterResource | Get-ClusterResourceDependency
     ForEach ($ClusterResourceDependency in $ClusterResourceDependencies) {
         If ($ClusterResourceDependency.DependencyExpression -ne ''){
             $String = 'Add-ClusterResourceDependency -Resource ' + [char]34 + $ClusterResourceDependency.resource + [char]34 + `
                 ' -Provider ' + [char]34 + $ClusterResourceDependency.DependencyExpression + [char]34
             Add-Content $cmdfilepath\$cmdfile $String -Encoding ASCII
         }
         }
 }
`

