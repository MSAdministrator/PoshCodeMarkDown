---
Author: josh einstein
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6269
Published Date: 2016-03-27t06
Archived Date: 2016-10-24t17
---

# get-dependency - 

## Description

##############################################################################

## Comments



## Usage



## TODO



## function

`get-pstoken`

## Code

`#
 #
 ##############################################################################
 #.AUTHOR
 ##############################################################################
 
 $BuiltInAliases   = @{}
 $BuiltInCmdlets   = @{}
 $BuiltInFunctions = @{}
 $BuiltInVariables = @{}
 
 ##############################################################################
 #.SYNOPSIS
 #
 #.DESCRIPTION
 #
 #.PARAMETER Type
 #
 #.PARAMETER Token
 #
 #.PARAMETER Text
 #
 #.PARAMETER Path
 #
 #.PARAMETER LiteralPath
 #
 #.PARAMETER Lines
 #
 #.EXAMPLE
 #
 #.LINK 
 #
 #.RETURNVALUE 
 ##############################################################################
 function Get-PSToken {
 
     [CmdletBinding(DefaultParameterSetName='Selection')]
     param(
         
         [Parameter(Position=1)]
         [String[]]$Type,
         
         [Parameter(Position=2)]
         [String[]]$Token,
 
         [Parameter(ParameterSetName='Path', Mandatory=$true, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
         [String[]]$Path,
         
         [Alias("PSPath")]
         [Parameter(ParameterSetName='LiteralPath', Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
         [String[]]$LiteralPath,
         
         [Parameter()]
         [Int32[]]$Lines
 
     )
 
     process {
     
         if ($PSCmdlet.ParameterSetName -eq 'Selection') {
             if (-not $PSISE) { throw 'The Selection parameter set is not valid outside of the PowerShell ISE.' }
             if (-not $PSISE.CurrentOpenedFile) { throw 'There is no file currently opened.' }
             if (-not $PSISE.CurrentOpenedFile.IsSaved) { throw 'Please save the currently active document first.' }
             if ($PSISE.CurrentOpenedFile.IsUntitled) { throw 'Please save the currently active document first.' }
             $ResolvedPaths = @(Resolve-Path -LiteralPath $PSISE.CurrentOpenedFile.FullPath)
         }
         elseif ($PSCmdlet.ParameterSetName -match '^(Literal)?Path$') {
             switch ($PSCmdlet.ParameterSetName) {
                 Path        { $ResolvedPaths = @(Resolve-Path -Path $Path) }
                 LiteralPath { $ResolvePathArgs = @(Resolve-Path -LiteralPath $LiteralPath) }
             }
         }
         
         foreach ($ResolvedPath in $ResolvedPaths) {
             
             $ScriptContent = Get-Content $ResolvedPath
             
             $ParserErrors = [System.Management.Automation.PSParseError[]]@()
             $ParserTokens = [System.Management.Automation.PSParser]::Tokenize($ScriptContent, [Ref]$ParserErrors)
             
             for ($i=0; $i -lt $ParserErrors.Length; $i++) {
                 $ParserError = $ParserErrors[$i]
                 if (-not $ParserError.Token) { Write-Warning $ParserError.Message }
                 else { Write-Warning "($($ParserError.Token.StartLine), $($ParserError.Token.StartColumn)) $($ParserError.Message)" }
             }
 
             if (-not $ParserTokens.Count) { return }
 
 
             if (-not $ParserTokens.Count) { return }
         
             $ParserTokens | Add-Member -PassThru NoteProperty Script (Split-Path -Leaf $ResolvedPath)
 
         }
 
 
 }
 
 
 ##############################################################################
 #.SYNOPSIS
 #
 #.DESCRIPTION
 #
 #.PARAMETER Path
 #
 #.PARAMETER LiteralPath
 #
 #.PARAMETER Text
 #
 #.PARAMETER Unresolved
 #
 #.PARAMETER Force
 #
 #.EXAMPLE
 #
 #.LINK 
 #
 #.RETURNVALUE 
 ##############################################################################
 function Get-Dependency {
 
     [CmdletBinding(DefaultParameterSetName='Selection')]
     param(
 
         [Parameter(ParameterSetName='Path', Position=1, Mandatory=$true, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
         [String[]]$Path,
         
         [Alias("PSPath")]
         [Parameter(ParameterSetName='LiteralPath', Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
         [String[]]$LiteralPath,
 
         [Parameter()]
         [Switch]$Unresolved,
         
         [Parameter()]
         [Switch]$Force
 
     )
 
     begin {
     
         $Dependencies = New-Object System.Collections.ArrayList
 
         $LocalFunctions    = @{}
         $ImportedAliases   = @{}
         $ImportedCmdlets   = @{}
         $ImportedFunctions = @{}
 
         ##############################################################################
         #.SYNOPSIS
         ##############################################################################
         filter Normalize-Identifier([String]$Name) {
             if ($_ -is [System.Management.Automation.PSToken]) { $Name = $_.Content }
             if ($_ -is [String]) { $Name = $_ }
             $Name -replace '^$?(Script|Global|Local):',''
         }
 
 
         ##############################################################################
         #.SYNOPSIS
         ##############################################################################
         function Get-PSTokenProxy([String[]]$Type,[String[]]$Token,[Int32[]]$Lines) {
 
             $GetPSTokenArgs = @{}
             if ($Type.Length)        { $GetPSTokenArgs['Type'] = $Type }
             if ($Token.Length)       { $GetPSTokenArgs['Token'] = $Token }
             if ($Lines.Length)       { $GetPSTokenArgs['Lines'] = $Lines }
 
             $ResolvedPaths | Get-PSToken @GetPSTokenArgs
 
         }
 
         ##############################################################################
         #.SYNOPSIS
         ##############################################################################
         function Discover-BuiltInCommands {
 
             if ($BuiltInCmdlets.Count -eq 0) {
 
                 $BuiltInAliases.Clear()
                 $BuiltInCmdlets.Clear()
                 $BuiltInFunctions.Clear()
                 $BuiltInVariables.Clear()
 
                 $Posh = [PowerShell]::Create()
 
                 try {
 
                     [Void]$Posh.AddScript('Get-Command -CommandType Cmdlet,Function,Alias')
                     foreach($CommandInfo in $Posh.Invoke()) {
                         switch($CommandInfo.CommandType) {
                             Alias    { $BuiltInAliases[$CommandInfo.Name]   = $true }
                             Cmdlet   { $BuiltInCmdlets[$CommandInfo.Name]   = $true }
                             Function { $BuiltInFunctions[$CommandInfo.Name] = $true }
                         }
                     }
 
                     [Void]$Posh.AddScript('Get-Variable')
                     foreach($VariableInfo in $Posh.Invoke()) {
                         $BuiltInVariables[$VariableInfo.Name] = $true
                     }
                     
                     $BuiltInVariables['this'] = $true
                     
                 }
                 finally {
                     $Posh.Dispose()
                 }
                 
             
         }
 
         ##############################################################################
         #.SYNOPSIS
         ##############################################################################
         function Discover-ModuleImports {
 
             $ImportedAliases.Clear()
             $ImportedCmdlets.Clear()
             $ImportedFunctions.Clear()
 
             $TokenLines = @(Get-PSTokenProxy Command Import-Module | %{$_.StartLine})
             
             $TokenNames = @(Get-PSTokenProxy CommandArgument -Lines $TokenLines | Normalize-Identifier)
 
             $MissingModules = @()
             foreach ($TokenName in $TokenNames) {
                 if (-not (Get-Module $TokenName)) {
                     $MissingModules += $TokenName
                 }
             }
             
             if ($MissingModules.Length) {
                 
                 $MenuItem1 = New-Object System.Management.Automation.Host.ChoiceDescription "&No - Do Not Import Modules"
                 $MenuItem2 = New-Object System.Management.Automation.Host.ChoiceDescription "&Yes - Import These Modules"
                 [System.Management.Automation.Host.ChoiceDescription[]]$MenuItems = @($MenuItem1,$MenuItem2)
                 
                 $MenuCaption = "Imported Modules Not Loaded"
                 $MenuMessage = "One or more modules imported by the script are not currently loaded.`r`n" +
                                "This can lead to unresolved external references and missing`r`n" +
                                "information in the dependency report.`r`n" +
                                "`r`n" +
                                "You can attempt to temporarily import the following modules into the`r`n" +
                                "session by selection the option below.`r`n" +
                                "`r`n"
 
                 foreach ($MissingModule in $MissingModules) {
                     $MenuMessage += "    Import-Module $MissingModule`r`n"
                 }
 
                 $Response = $Host.UI.PromptForChoice($MenuCaption, $MenuMessage, $MenuItems, 0)
                 if ($Response = 1) {
                     foreach ($MissingModule in $MissingModules) {
                         Invoke-Expression ".{Import-Module $MissingModule -ErrorAction Continue}"
                     }
                 }
                 
             }
                 
 
             $ImportedModules = @(Get-Module $TokenNames -ListAvailable) + @(Get-Module $TokenNames)
             foreach ($ImportedModule in $ImportedModules) {
                 $ImportedModule.ExportedAliases.Values   | %{ $ImportedAliases[$_.Name]   = $_ }
                 $ImportedModule.ExportedCmdlets.Values   | %{ $ImportedCmdlets[$_.Name]   = $_ }
                 $ImportedModule.ExportedFunctions.Values | %{ $ImportedFunctions[$_.Name] = $_ }
             }
 
         }
 
         ##############################################################################
         #.SYNOPSIS
         ##############################################################################
         function Discover-LocalFunctions {
             
             $LocalFunctions.Clear()
             
             $TokenLines = @(Get-PSTokenProxy Keyword function,filter | %{$_.StartLine})
 
             $TokenNames = @(Get-PSTokenProxy CommandArgument -Lines $TokenLines | Normalize-Identifier)
 
             foreach ($TokenName in $TokenNames) {
                 $LocalFunctions[$TokenName] = $true
             }
             
         }
 
         ##############################################################################
         #.SYNOPSIS
         ##############################################################################
         filter Exclude-BuiltInReferences {
             
             if ($Force) { return $_ }
 
             $Token = $_
             $TokenName = Normalize-Identifier $Token.Content
 
             if ($Token.Type -eq 'Command') {
                 if ($BuiltInAliases[$TokenName]) { return }
                 if ($BuiltInCmdlets[$TokenName]) { return }
                 if ($BuiltInFunctions[$TokenName]) { return }
             }
             elseif ($Token.Type -eq 'Variable') {
                 if ($BuiltInVariables[$TokenName]) { return }
             }
             
             $Token
 
         }
 
         ##############################################################################
         #.SYNOPSIS
         ##############################################################################
         filter Exclude-LocalReferences {
 
             if ($Force) { return $_ }
 
             $Token = $_
             $TokenName = Normalize-Identifier $Token.Content
 
             if ($Token.Type -eq 'Command') {
                 if ($LocalFunctions[$TokenName]) { return }
             }
             
             $Token
 
         }
 
         ##############################################################################
         #.SYNOPSIS
         ##############################################################################
         function Get-SafeCommand([String]$Name) {
 
             $SafeName = [System.Management.Automation.WildcardPattern]::Escape($Name)
             
             $CommandInfo = Get-Command -Name $SafeName -ErrorAction SilentlyContinue | Select -First 1
             if (-not $CommandInfo) { $CommandInfo = $ImportedAliases[$Name] }
             if (-not $CommandInfo) { $CommandInfo = $ImportedFunctions[$Name] }
             if (-not $CommandInfo) { $CommandInfo = $ImportedCmdlets[$Name] }
 
             $CommandInfo
 
         }
 
 
         ##############################################################################
         #.SYNOPSIS
         ##############################################################################
         function Get-SafeVariable([String]$Name) {
             $SafeName = [System.Management.Automation.WildcardPattern]::Escape($Name)
             $VariableInfo = Get-Variable -Name $SafeName -Scope Global -ErrorAction SilentlyContinue | Select -First 1
             $VariableInfo            
         }
 
         ##############################################################################
         #.SYNOPSIS
         #
         #.DESCRIPTION
         ##############################################################################
         function Select-UniqueTokens {
 
             $Input | 
                 Group-Object Type,Content |
                 Select-Object @(
                     @{N='Type';      E={ $_.Group[0].Type      } },
                     @{N='Content';   E={ $_.Group[0].Content   } },
                     @{N='Script';    E={ $_.Group[0].Script    } }
                     @{N='StartLine'; E={ $_.Group[0].StartLine } }
                  )
 
         }
 
         ##############################################################################
         #.SYNOPSIS
         ##############################################################################
         function New-Dependency($Token) {
 
             New-Object PSObject |
                 Add-Member -PassThru NoteProperty Script ($Token.Script) |
                 Add-Member -PassThru NoteProperty Module ($null) |
                 Add-Member -PassThru NoteProperty Type ($Token.Type) |
                 Add-Member -PassThru NoteProperty Name (Normalize-Identifier $Token.Content) |
                 Add-Member -PassThru NoteProperty Target ($null) |
                 Add-Member -PassThru NoteProperty Resolved ($false) |
                 Add-Member -PassThru NoteProperty File ($null)
 
         }
     
 
     process {
 
         if ($PSCmdlet.ParameterSetName -eq 'Selection') {
             if (-not $PSISE) { throw 'The Selection parameter set is not valid outside of the PowerShell ISE.' }
             if (-not $PSISE.CurrentOpenedFile) { throw 'There is no file currently opened.' }
             if (-not $PSISE.CurrentOpenedFile.IsSaved) { throw 'Please save the currently active document first.' }
             if ($PSISE.CurrentOpenedFile.IsUntitled) { throw 'Please save the currently active document first.' }
             $ResolvedPaths = @(Resolve-Path -LiteralPath $PSISE.CurrentOpenedFile.FullPath)
         }
         elseif ($PSCmdlet.ParameterSetName -match '^(Literal)?Path$') {
             switch ($PSCmdlet.ParameterSetName) {
                 Path        { $ResolvedPaths = @(Resolve-Path -Path $Path) }
                 LiteralPath { $ResolvedPaths = @(Resolve-Path -LiteralPath $LiteralPath) }
             }
         }        
         
         Discover-BuiltInCommands
         Discover-ModuleImports
         Discover-LocalFunctions
         
         
         $Tokens = @(
             Get-PSTokenProxy Command,Variable |
                 Exclude-BuiltInReferences |
                 Exclude-LocalReferences |
                 Select-UniqueTokens
         )
         
         
         foreach ($Token in $Tokens) {
 
             $Dependency = New-Dependency $Token
 
             if ($Token.Type -eq 'Variable') {
 
                 if ($VariableInfo = Get-SafeVariable $Dependency.Name) {
 
                     $Dependency.Module = $VariableInfo.ModuleName
                     $Dependency.Resolved = $true
 
                 }
                 else {
                 
                     
                     continue
 
                 }
 
             
 
             if ($Token.Type -eq 'Command') {
                 
                 if ($CommandInfo = Get-SafeCommand $Dependency.Name) {
 
                     $Dependency.Type= $CommandInfo.CommandType
                     $Dependency.Resolved = $true
 
                     if ($CommandInfo.CommandType -eq 'Alias') {
                         
                         if ($CommandInfo.ResolvedCommandName) {
                         
                             if ($CommandInfo.Name -eq $CommandInfo.ResolvedCommandName) {
                                 if ($BuiltInAliases[$CommandInfo.ResolvedCommandName]) { continue }
                                 if ($BuiltInCmdlets[$CommandInfo.ResolvedCommandName]) { continue }
                                 if ($BuiltInFunctions[$CommandInfo.ResolvedCommandName]) { continue }
                             }
 
                             $Dependency.Target = $CommandInfo.ResolvedCommandName
                             $CommandInfo = $CommandInfo.ResolvedCommand
                             
                         }
                         else {
 
                             $Dependency.Resolved = $false
                             
                         }
                         
                     
                     if ($CommandInfo.ModuleName) { $Dependency.Module = $CommandInfo.ModuleName }
                     if ($CommandInfo.ScriptBlock.File) { $Dependency.File = $CommandInfo.ScriptBlock.File }
                     if ($CommandInfo.Path) { $Dependency.File = $CommandInfo.Path }
                     if ($CommandInfo.DLL) { $Dependency.File = $CommandInfo.DLL }
                 
                     
 
             if ($Unresolved -and $Dependency.Resolved) { continue }
 
             [Void]$Dependencies.Add($Dependency)
             
 
     
     end {
     
         $Dependencies | Sort Script,Module,Type,Name
     
     
 }
`

