---
Author: geoffrey guynn
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3186
Published Date: 2014-01-25t08
Archived Date: 2014-03-10t19
---

# get-software function - 

## Description

i was having a great deal of difficulty in properly enumerating software applications on remote machines, so i wrote this function to help ease the problem somewhat. among the many methods of gathering this information that are available, i had the most success using the “software\\microsoft\\windows\\currentversion\\uninstall” registry key as an information source. in order to use this function, the powershell instance must support .net 4.0 or greater, which is fairly straightforward if you follow these instructions.

## Comments



## Usage



## TODO



## function

`get-software`

## Code

`#
 #
 Function Get-Software{
     <#
     .SYNOPSIS
         Gets the software applications on a remote computer.
     .DESCRIPTION
         This function interrogates the remote registry for installed software products.
         It then returns an array of powershell objects that can be sorted and parsed.
         
         Optionally, you can provide a product name that will filter applications based on
         displayname before they are returned, thus reducing the typing needed to get specific results.
     .PARAMETER computer
         The name of the computer to retrieve a software list from.
     .PARAMETER product
         The partial name of a software application to search for.
     .INPUTS
     .OUTPUTS
     .NOTES
         Name: Get-Software
         Author: Geoffrey Guynn
         DateCreated 9 Aug 2011
     .EXAMPLE
         Get-Software -computer "computer" -name "Adobe"
         Returns all instances of Adobe software on the computer.
     #>
 
     [cmdletbinding(
         SupportsShouldProcess = $True,
         DefaultParameterSetName = "process",
         ConfirmImpact = "low"
     )]
     param(
         [ValidateNotNullOrEmpty()]
         [string]$Computer
         ,
         [string]$Product
     )
     $OSArch = (Gwmi -computer $Computer win32_operatingSystem).OSArchitecture
     if ($OSArch -like "*64*") {$Architectures = @("32bit","64bit")}
     else {$Architectures = @("32bit")}
     $arApplications = @()
     foreach ($Architecture in $Architectures){
         if ($Architecture -like "*64*"){
             $strSubKey = "SOFTWARE\\WOW6432Node\\Microsoft\\Windows\\CurrentVersion\\Uninstall"
             $SoftArchitecture = "32bit"
             $RegViewEnum = [Microsoft.Win32.RegistryView]::Registry64
         }
         elseif ($Architectures -notcontains "64bit"){
             $strSubKey = "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall"
             $SoftArchitecture = "32bit"
             $RegViewEnum = [Microsoft.Win32.RegistryView]::Registry32
         }
         else{
             $strSubKey = "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall"
             $SoftArchitecture = "64bit"
             $RegViewEnum = [Microsoft.Win32.RegistryView]::Registry64
         }
         #************************************************************************
         $remRegistry = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', $Computer, $RegViewEnum)
         $remKey = $remRegistry.OpenSubKey($strSubKey)
         $remSoftware = $remKey.GetSubKeyNames()
         foreach ($remApplication in $RemSoftware){
             $remProgram = $remRegistry.OpenSubKey("$strSubKey\\$remApplication")
             $strDisplayName = $remProgram.GetValue("DisplayName")
             if ($strDisplayName){
                 $arProperties = $remProgram.GetValueNames()
                 $objApplication = New-Object System.Object
                 foreach ($strProperty in $arProperties){
                     $strValue = [string]($remProgram.GetValue($strProperty))
                     if ($strValue){
                         if (!$strProperty){
                             $objApplication | Add-Member -type NoteProperty -Name "(Default)" -Value $strValue
                         }
                         else{
                             $objApplication | Add-Member -type NoteProperty -Name ([string]$strProperty) -Value $strValue
                         }
                     }
                 }
                 $objApplication | Add-Member -type NoteProperty -Name "Architecture" -Value $SoftArchitecture
                 $arApplications += $objApplication
             }
         }
     }
     if ($Product){
         $objApplication = $arApplications | ? {$_.DisplayName -Like "*$Product*"} | Sort-Object -property Architecture,DisplayName
         return $objApplication
     }
     $arApplications = $arApplications | Sort-Object -property Architecture,DisplayName
     return $arApplications
 }
`

