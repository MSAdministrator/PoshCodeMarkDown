---
Author: lee holmes
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2212
Published Date: 2011-09-09t21
Archived Date: 2016-03-18t21
---

# search-wminamespace.ps1 - 

## Description

from windows powershell cookbook (o’reilly) by lee holmes

## Comments



## Usage



## TODO



## class

`new-wmimatch`

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
 
 Search the WMI classes installed on the system for the provided match text.
 
 .EXAMPLE
 
 Search-WmiNamespace Registry
 Searches WMI for any classes or descriptions that mention "Registry"
 
 .EXAMPLE
 
 Search-WmiNamespace Process ClassName,PropertyName
 Searchs WMI for any classes or properties that mention "Process"
 
 .EXAMPLE
 
 Search-WmiNamespace CPU -Detailed
 Searches WMI for any class names, descriptions, or properties that mention
 "CPU"
 
 #>
 
 param(
     [Parameter(Mandatory = $true)]
     [string] $Pattern,
 
     [switch] $Detailed,
 
     [switch] $Full,
 
     [string[]] $MatchOptions = ("ClassName","ClassDescription")
 )
 
 Set-StrictMode -Off
 
 function New-WmiMatch
 {
     param( $matchType, $className, $propertyName, $line )
 
     $wmiMatch = New-Object PsObject -Property @{
         MatchType = $matchType;
         ClassName = $className;
         PropertyName = $propertyName;
         Line = $line
     }
 
     $wmiMatch
 }
 
 if($detailed)
 {
     $matchOptions = "ClassName","ClassDescription","PropertyName"
 }
 
 if($full)
 {
     $matchOptions =
         "ClassName","ClassDescription","PropertyName","PropertyDescription"
 }
 
 foreach($matchOption in $matchOptions)
 {
     $fullMatchOptions =
         "ClassName","ClassDescription","PropertyName","PropertyDescription"
 
     if($fullMatchOptions -notcontains $matchOption)
     {
         $error = "Cannot convert value {0} to a match option. " +
             "Specify one of the following values and try again. " +
             "The possible values are ""{1}""."
         $ofs = ", "
         throw ($error -f $matchOption, ([string] $fullMatchOptions))
     }
 }
 
 foreach($class in Get-WmiObject -List -Rec)
 {
     $managementOptions = New-Object System.Management.ObjectGetOptions
     $managementOptions.UseAmendedQualifiers = $true
     $managementClass =
         New-Object Management.ManagementClass $class.Name,$managementOptions
 
     if($matchOptions -contains "ClassName")
     {
         if($managementClass.Name -match $pattern)
         {
             New-WmiMatch "ClassName" `
                 $managementClass.Name $null $managementClass.__PATH
         }
     }
 
     if($matchOptions -contains "ClassDescription")
     {
         $description =
             $managementClass.Qualifiers |
                 foreach { if($_.Name -eq "Description") { $_.Value } }
         if($description -match $pattern)
         {
             New-WmiMatch "ClassDescription" `
                 $managementClass.Name $null $description
         }
     }
 
     foreach($property in $managementClass.Properties)
     {
         if($matchOptions -contains "PropertyName")
         {
             if($property.Name -match $pattern)
             {
                 New-WmiMatch "PropertyName" `
                     $managementClass.Name $property.Name $property.Name
             }
         }
 
         if($matchOptions -contains "PropertyDescription")
         {
             $propertyDescription =
                 $property.Qualifiers |
                     foreach { if($_.Name -eq "Description") { $_.Value } }
             if($propertyDescription -match $pattern)
             {
                 New-WmiMatch "PropertyDescription" `
                     $managementClass.Name $property.Name $propertyDescription
             }
         }
     }
 }
`

