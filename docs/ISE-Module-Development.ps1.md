---
Author: jrich523
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6893
Published Date: 2017-05-12t19
Archived Date: 2017-05-16t10
---

# ise module development - 

## Description

ise profile code for doing module development

## Comments



## Usage



## TODO



## function

`import-currentfileasmodule`

## Code

`#
 #
   Function Import-CurrentFileAsModule
 {
     [cmdletbinding()]
     param()
     $filePath = $psise.CurrentFile.FullPath
     $folder = split-path $filePath
     if($psise.CurrentFile.IsUntitled){Write-Error "Must save file first! Sorry didn't feel like implementing the dialog box!" -ErrorAction Stop}
     if(-not $psise.CurrentFile.IsSaved){$psise.CurrentFile.Save()}
     $global:WorkingModule = $null
     try{ $Global:WorkingModule = Import-Module $folder -Force -ErrorAction Stop -PassThru -Verbose:$false | select -ExpandProperty name}
     catch{$folderFailed = $true}
     
     if($folderFailed)
     {
         try {Import-Module $filePath -Force -ErrorAction Stop -Verbose:$false}
         catch{ write-error "Not a module file!" -ErrorAction Stop}
     } 
     if(Test-Path function:\PostModuleProcess)
     {
         Write-Verbose "Processing PostModuleProcess Function"
         PostModuleProcess
     }
     else
     {
         Write-Verbose "--Create a PostModuleProcess function to excute code after import--"
     }
 }
 
 Function Test-Module
 {
     $filePath = $psise.CurrentFile.FullPath
     $folder = split-path $filePath
     if($psise.CurrentFile.IsUntitled){Write-Error "Must save file first! Sorry didn't feel like implementing the dialog box!" -ErrorAction Stop}
     if(-not $psise.CurrentFile.IsSaved){$psise.CurrentFile.Save()}
 
     $cmd = "cd d:\ps;ipmo $folder"
 
     if(Test-Path function:\PostModuleProcess)
     {
         Write-Verbose "Processing PostModuleProcess Function"
         $cmd += ";" + (gi function:\PostModuleProcess).Definition
     }
 
     start powershell -ArgumentList "-noprofile -noexit -command $cmd"
 }
 Function Get-ModuleVariable{
 param($Name)
     if($global:workingmodule)
     {
         if($name)
         {
             &(gmo $global:workingmodule){Get-Variable -Name $args[0] -Scope script -ValueOnly} $name
         }
         else
         {
             &(gmo $global:workingmodule){Get-Variable -Scope script -ValueOnly}
         }
     }
 }
 Function Set-ModuleVariable{
 [cmdletbinding()]
 param(
     [parameter(manditory=$true)]
     $Name,
     [parameter(manditory=$true)]
     $Value)
     if($global:workingmodule)
     {
         &(gmo $global:workingmodule){Set-Variable -Name $args[0] -Value $args[1] -Scope script} $Name $Value
     }
 }
 $null=$psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add('Reload Module',{Import-CurrentFileAsModule -verbose},"F6")
 $null=$psISE.CurrentPowerShellTab.AddOnsMenu.Submenus.Add('Test Module',{Test-Module},"Shift+F6")
`

