---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.0
Encoding: ascii
License: cc0
PoshCode ID: 4379
Published Date: 2013-08-11t06
Archived Date: 2013-08-13t09
---

# new-module - 

## Description

generate a new powershell module from a few scripts.

## Comments



## Usage



## TODO



## function

`new-module`

## Code

`#
 #
 function New-Module {
    #.Synopsis
    #.Description
    #
    #
    #.Example
    #
    #.Example
    #
    #.Example
    #
    #
    #.Example
    #
    [CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact="Medium", DefaultParameterSetName="NewModuleManifest")]
    param(
       [Switch]$Force,
 
       [Switch]$Upgrade,
 
       [Parameter(Position=0, Mandatory=$true)]
       [ValidateScript({if($_ -match "[$([regex]::Escape(([io.path]::GetInvalidFileNameChars() -join '')))]") { throw "The ModuleName must be a valid folder name. The character '$($matches[0])' is not valid in a Module name."} else { $true } })]
       $ModuleName,
 
       [Parameter(Position=1, Mandatory=$true, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName="OverwriteModule")]
       [Alias("PSPath")]
       $Path,
 
       [Switch]$Recurse,
 
       [Switch]$StrictVerbs,
 
       [PSDefaultValue(Help = {"Env:UserName: (${Env:UserName})"})]
       [String]$Author = $Env:UserName,
 
       [Parameter(Position=1)]
       [PSDefaultValue(Help = {"'A collection of script files by ${Env:UserName}' (uses the value from the Author parmeter)"})]
       [string]${Description} = "A collection of script files by $Author",
 
       [Parameter()]
       [PSDefaultValue(Help = "1.0 (when -Upgrade is set, increments the existing value to the nearest major version number)")]
       [Alias("Version","MV")]
       [Version]${ModuleVersion} = "1.0",
 
       [AllowEmptyString()]
       [String]$CompanyName = "None (Personal Module)",
 
       [version]
       [PSDefaultValue(Help = {"Your current CLRVersion number (rounded): ($($PSVersionTable.CLRVersion.ToString(2)))"})]
       ${ClrVersion} = $($PSVersionTable.CLRVersion.ToString(2)),
 
       [version]
       [PSDefaultValue(Help = {"Your current PSVersion number (rounded): ($($PSVersionTable.PSVersion.ToString(2))"})]
       [Alias("PSV")]
       ${PowerShellVersion} = $("{0:F1}" -f (([Double]$PSVersionTable.PSVersion.ToString(2)) - 1.0)),
       
       [System.Object[]]
       [Alias("Modules","RM")]
       ${RequiredModules} = $null,
       
       [AllowEmptyCollection()]
       [string[]]
       [Alias("Assemblies","RA")]
       ${RequiredAssemblies} = $null
    )
 
    begin {
       if(Test-Path $ModuleName -Type Container) {
          [String]$ModulePath = Resolve-Path $ModuleName
       } else {
          if(!$ModuleName.Contains([io.path]::DirectorySeparatorChar) -and !$ModuleName.Contains([io.path]::AltDirectorySeparatorChar)) {
             [String]$ModulePath = Join-Path ([Environment]::GetFolderPath("MyDocuments")) "WindowsPowerShell\Modules\$ModuleName"
          } else {
             [String]$ModulePath = $ModuleName
          }
       }
       [String]$ModuleName = Split-Path $ModuleName -Leaf
 
       if($Path) {
          $Path = switch($Path) {
             {$_ -is [String]} { 
                if(Test-Path $_) {
                   Get-ChildItem $_ -Recurse:$Recurse | % { $_.FullName }
                } else { throw "Can't find the file '$_' doesn't exist." }
             }
             {$_ -is [IO.FileSystemInfo]} { $_.FullName }
             default { Write-Warning $_.GetType().FullName + " type not supported for `$Path" }
          }
       }
 
       $ScriptFiles = @()
    }
 
    process {
       $ScriptFiles += switch($Path){
          {$_ -is [String]} { 
             if(Test-Path $_) {
                Resolve-Path $_ | % { $_.ProviderPath }
             } else { throw "Can't find the file '$_' doesn't exist." }
          }
          {$_ -is [IO.FileSystemInfo]} { $_.FullName }
          default { Write-Warning $_.GetType().FullName + " type not supported for `$Path" }
       }
    }
 
    end {
       $ErrorActionPreference = "Stop"
 
       if($ScriptFiles) {
          if(!$Upgrade -and (Test-Path $ModulePath)) {
             if($Force -Or $PSCmdlet.ShouldContinue("The specified Module already exists: '$ModulePath'. Do you want to delete it and start over?", "Deleting '$ModulePath'")) {
                Remove-Item $ModulePath -recurse
             } else {
                throw "The specified ModuleName '$ModuleName' already exists in '$ModulePath'. Please choose a new name, or specify -Force to overwrite that folder."
             }
          }
 
          if(!$Upgrade -or !(Test-Path $ModulePath)) {
             try {
                $null = New-Item -Type Directory $ModulePath
             } catch [Exception]{
                Write-Error "Cannot create Module Directory: '$ModulePath' $_"
             }
          }
 
          foreach($file in Get-Item $ScriptFiles | Where { !$_.PSIsContainer }) {
             $Destination = Join-Path $ModulePath (Resolve-Path $file -Relative )
             if(!(Test-Path (Split-Path $Destination))) {
                $null = New-Item -Type Directory (Split-Path $Destination)
             }
             Copy-Item $file $Destination
          }
       }
 
       Push-Location $ModulePath
 
       try {
          if(!$Upgrade -Or !(Test-Path "${ModuleName}.psm1")) {
             if($Force -Or !(Test-Path "${ModuleName}.psm1") -or $PSCmdlet.ShouldContinue("The specified '${ModuleName}.psm1' already exists in '$ModulePath'. Do you want to overwrite it?", "Creating new '${ModuleName}.psm1'")) {
                Set-Content "${ModuleName}.psm1" @'
 Push-Location $PSScriptRoot
 Import-LocalizedData -BindingVariable ModuleManifest
 $ModuleManifest.FileList -like "*.ps1" | % {
     $Script = Resolve-Path $_ -Relative
     Write-Verbose "Loading $Script"
     . $Script
 }
 Pop-Location
 '@
             } else {
                throw "The specified Module '${ModuleName}.psm1' already exists in '$ModulePath'. Please create a new Module, or specify -Force to overwrite the existing one."
             }
          }
 
 
          if($Force -Or $Upgrade -or !(Test-Path "${ModuleName}.psd1") -or $PSCmdlet.ShouldContinue("The specified '${ModuleName}.psd1' already exists in '$ModulePath'. Do you want to update it?", "Creating new '${ModuleName}.psd1'")) {
             if($Upgrade -and (Test-Path "${ModuleName}.psd1")) {
                Import-LocalizedData -BindingVariable ModuleInfo -File "${ModuleName}.psd1" -BaseDirectory $ModulePath
             } else {
                $ModuleInfo = @{
                   ModuleVersion = 0.0
                   Author = $Author
                   Description = $Description
                   CompanyName = $CompanyName
                   ClrVersion = $ClrVersion
                   PowerShellVersion = $PowerShellVersion
                   RequiredModules = $RequiredModules
                   RequiredAssemblies = $RequiredAssemblies
                }
             }
 
             $FileList = Get-ChildItem -Recurse | Where { !$_.PSIsContainer } | Resolve-Path -Relative
 
             $verbs = if($Strict){ Get-Verb | % { $_.Verb +"-*" } } else { "*-*" }
 
             $ModuleInfo.Path = Resolve-Path "${ModuleName}.psd1"
             $ModuleInfo.RootModule = "${ModuleName}.psm1"
             $ModuleInfo.ModuleVersion = [Math]::Floor(1.0 + $ModuleInfo.ModuleVersion).ToString("F1")
             $ModuleInfo.FileList = $FileList
             $ModuleInfo.TypesToProcess = $FileList -match ".*Types?\.ps1xml"
             $ModuleInfo.FormatsToProcess = $FileList -match ".*Formats?\.ps1xml"
             $ModuleInfo.NestedModules = $FileList -like "*.psm1" -notlike "*${ModuleName}.psm1"
             $ModuleInfo.FunctionsToExport = $Verbs
             $ModuleInfo.AliasesToExport = "*"
             $ModuleInfo.VariablesToExport = "${ModuleName}*"
             $ModuleInfo.CmdletsToExport = $Null
 
             $null = $PSBoundParameters.Remove("Path")
             $null = $PSBoundParameters.Remove("Force")
             $null = $PSBoundParameters.Remove("Upgrade")
             $null = $PSBoundParameters.Remove("Recurse")
             $null = $PSBoundParameters.Remove("ModuleName")
             foreach($key in $PSBoundParameters.Keys) {
                $ModuleInfo.$key = $PSBoundParameters.$key
             }
 
             New-ModuleManifest @ModuleInfo
             Get-Item $ModulePath
          }  else {
             throw "The specified Module Manifest '${ModuleName}.psd1' already exists in '$ModulePath'. Please create a new Module, or specify -Force to overwrite the existing one."
          }
       } catch {
          throw
       } finally {
          Pop-Location
       }
    }
 }
`

