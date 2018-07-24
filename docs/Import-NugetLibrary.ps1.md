---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 4.0
Encoding: ascii
License: cc0
PoshCode ID: 6300
Published Date: 2016-04-12t03
Archived Date: 2016-06-17t08
---

# import-nugetlibrary - 

## Description

(install and) load the assemblies from a nuget package into powershell

## Comments



## Usage



## TODO



## module

`import-nugetlibrary`

## Code

`#
 #
 function Import-NugetLibrary {
     param(
         [Parameter(Mandatory)]
         [String]$Name,
 
         [String]$Destination = "$(Split-Path $Profile)\Libraries"
     )
     $ErrorActionPreference = "Stop"
 
     if(!(Test-Path $Destination -Type Container)) {
         throw "The destination path ($Destination) must point to an existing folder, NuGet will install packages in subdirectories of this folder."
     }
 
     $Destination = (Convert-Path $Destination).TrimEnd("\")
     $Package = Install-Package -Name $Name -Destination $Destination -ProviderName NuGet -Source 'https://www.nuget.org/api/v2' -ExcludeVersion -PackageSaveMode nuspec -Force
     $PackagePath = "$Destination\$Name"
 
     if($PSVersionTable.PSVersion -ge "4.0") {
         $Versions = "net46","net452","net451","net45","net40","net35","net20"
     }elseif($PSVersionTable.PSVersion -ge "3.0") {
         $Versions = "net40","net35","net20","net45","net451","net452","net46"
     }else {
         $Versions = "net35","net20"
     }
 
     $LibraryPath = ($Versions -replace "^", "$PackagePath\lib\") + "$PackagePath\lib\" + "$PackagePath"   | 
         Convert-Path -ErrorAction SilentlyContinue | Select -First 1
 
 
     $Number = $LibraryPath -replace '.*?([\d\.]+)','$1' -replace '(?<=\d)(?=\d)','.'
 
 
     $References = ([xml](gc "$PackagePath\$Name.nuspec")).package.metadata.references
 
     if(!$References) {
         $Assemblies = Get-ChildItem $LibraryPath -Filter *.dll
     } else {
         $group = $references.Group | where { $_.targetFramework.EndsWith($number) }
         if($group) {
             $Files = $group.reference.File 
         } else {
             $Files = @($references.SelectNodes("//*[name(.) = 'reference']").File | Select -Unique)
         }
 
         $Assemblies = Get-Item ($Files -replace "^", "$LibraryPath\")
     }
 
     Push-Location $Destination
     for($e=0; $e -lt $Assemblies.Count; $e++) {
         $success = $true
         foreach($assm in $Assemblies) {
             Write-Verbose "Import Library $(Resolve-Path $Assm.FullName -Relative)"
             Add-Type -Path $Assm.FullName -ErrorAction SilentlyContinue -ErrorVariable failure
             if($failure) {
                 $success = $false
             } else {
                 Write-Host "LOADING: " -Fore Cyan -NoNewline
                 Write-Host $LibraryPath\ -Fore Yellow -NoNewline
                 Write-Host $Assm.Name -Fore White
             }
         }
         if($success) { break }
     }
     Pop-Location
     if(!$success) { throw $failure }
     return
 }
`

