---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.1
Encoding: ascii
License: cc0
PoshCode ID: 3930
Published Date: 2013-02-01t23
Archived Date: 2013-09-16t05
---

# helpmodules - 

## Description

a module to let you fetch and read help for modules you don’t have!  now get-modulehelp works without specifying the modulename, and you can easily fetch help for modules from remote servers.

## Comments



## Usage



## TODO



## module

`new-helpmodule`

## Code

`#
 #
 $PSModuleHelpRoot = Convert-Path "~\Documents\WindowsPowerShellModuleHelp"
 
 function New-HelpModule {
   #.Synopsis
   #.Example
   #
   #.Example
   #
   #.Example
   #
   [CmdletBinding(DefaultParameterSetName="ModuleInfo")]
   param(
     [Parameter(ValueFromPipelineByPropertyName=$true, Mandatory=$true, Position=0)]
     [Alias("Name")]
     [String]$ModuleName,
 
     [Parameter(ValueFromPipelineByPropertyName=$true, Mandatory=$true, Position=1)]
     [String]$Version,
 
     [Parameter(ValueFromPipelineByPropertyName=$true, Mandatory=$true, Position=2)]
     [Guid]$Guid,
 
     [Parameter(ValueFromPipelineByPropertyName=$true, Mandatory=$true, Position=3)]
     [String]$HelpInfoUri,
 
     #
     [Parameter(ValueFromPipelineByPropertyName=$true)]
     $ExportedCommands,
 
     [String]$ModuleHelpRoot = $PSModuleHelpRoot,
 
     [Alias("Culture","PSCulture")]
     [CultureInfo[]]$HelpCulture = ${PSUICulture},
 
     #
     [Switch]$StubFunctions
   )
   begin {
     $ModulesToUpdate = @()
   }
   process {
     $ModuleDir = mkdir ${ModuleHelpRoot}\${ModuleName}\ -Force 
 
     if($ExportedCommands -is [System.Collections.Generic.Dictionary[System.String,System.Management.Automation.CommandInfo]]) {
       [string[]]$ExportedCommands = $ExportedCommands.Keys
     } 
 
     New-ModuleManifest -Path ${ModuleHelpRoot}\${ModuleName}\${ModuleName}.psd1 `
       -Guid $Guid -HelpInfoUri $HelpInfoUri -ModuleVersion $Version `
       -FunctionsToExport $ExportedCommands `
       -RootModule "Blank.psm1"
 
     $ModulesToUpdate += $ModuleName
 
     if($StubFunctions) {
       Remove-Item "${ModuleHelpRoot}\${ModuleName}\Blank.psm1" -ErrorAction SilentlyContinue
       foreach($name in $ExportedCommands) {
         Add-Content "${ModuleHelpRoot}\${ModuleName}\Blank.psm1" "#.ExternalHelp *.xml `nfunction $name {}`n"
       }
     }
   }
   end {
     PowerShell -NoProfile -Command "&{ `$Env:PSModulePath = '$ModuleHelpRoot'; Update-Help -Module '$($ModulesToUpdate -join "','")' -UICulture '$($HelpCulture -join "','")'}"
   }
 }
 
 function Get-ModuleHelp {
   #.Synopsis
   [CmdletBinding(DefaultParameterSetName="MamlCommandHelpInfo")]
   param(
     [Alias("Name")]
     [Parameter(Mandatory=$true, Position = 0, ValueFromPipelineByPropertyName = $true)]
     $CommandName,
     [Parameter(Position = 1, ValueFromPipelineByPropertyName = $true)]
     $ModuleName = "*",
 
     $ModuleHelpRoot = $PSModuleHelpRoot,
 
     [String]$Parameter,
 
     [Switch]$Examples,
 
     [Switch]$Full,
 
     [Switch]$Detailed,
 
     [Alias("Culture","PSCulture")]
     [CultureInfo]$HelpCulture = ${PSUICulture}
   )
   process {
     Write-Verbose "Culture: $HelpCulture HelpSet: $($PSCmdlet.ParameterSetName)"
     $matched = $false
     foreach($node in Select-Xml "//*[local-name() = 'details']/*[local-name() = 'name' and text() = '$CommandName']/../.." -Path ${ModuleHelpRoot}\${ModuleName}\${PSUICulture}\*.xml | Select-Object -Expand Node) {
       if($Parameter) {
         foreach($param in $node.parameters.parameter) {
           if($param.name -like $Parameter) {
             $matched = $true
             $param | FixMaml -Type $($PSCmdlet.ParameterSetName)
           }
         }
         if(!$matched) {
           throw "No parameter matches criteria $Parameter" 
         }
       } else {
         $matched = $true
         $node | FixMaml -Type $($PSCmdlet.ParameterSetName)
       }
     }
   }
 }
 
 function FixMaml {
   #.Synopsis
   [CmdletBinding()]
   param(
     [Parameter(ValueFromPipeline=$true)]
     $Node,
 
     [Parameter(Mandatory=$true)]
     $Type = "MamlCommandHelpInfo"
   )
   process {
     $node.PSTypeNames.Clear()
     $node.PSTypeNames.Add($type)       
     if($node.description) {
       Add-Member -Input $node NoteProperty description @(
         $Node.RemoveChild($node.description).para | % {
           $p = New-Object PSObject -Property @{ Text = $_ };
           $p.PSTypeNames.Clear(); 
           $p.PSTypeNames.Add("MamlParaTextItem"); 
           $p
         } )
     }
     if($node.details) {
       $null = $node.details | FixMaml -Type $Type
     } 
     Write-Output $node   
   }
 }
 
 Export-ModuleMember -Function New-HelpModule, Get-ModuleHelp -Variable PSModuleHelpRoot
 
`

