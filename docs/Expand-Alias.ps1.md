---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.3
Encoding: ascii
License: cc0
PoshCode ID: 4209
Published Date: 2013-06-19t18
Archived Date: 2013-06-25t00
---

# expand-alias - 

## Description

converts aliases and parameter shortcuts in scripts to make them more portable.  now resolves command names to include the module (make sure you load modules you need), resolves parameter aliases, etc. (works in powershell 3 ctp1 too).

## Comments



## Usage



## TODO



## module

`resolve-command`

## Code

`#
 #
 ########################################################################################################################
 ########################################################################################################################
 Set-StrictMode -Version latest
 function Resolve-Command {
   #.Synopsis
   [CmdletBinding()]
   param( 
     [Parameter(Mandatory=$true)]
     [String]$Name, 
 
     [String[]]$AllowedModule=$(@(Microsoft.PowerShell.Core\Get-Module | Select -Expand Name) + 'Microsoft.PowerShell.Core'),
 
     [Parameter()]
     [string[]]$AllowedCommand
   )
   end {
     $Search = $Name -replace '(.)$','[$1]'
     $Commands = @(Microsoft.PowerShell.Core\Get-Command $Search -Module $AllowedModule -ErrorAction SilentlyContinue)
     if(!$Commands) {
       if($match = $AllowedCommand -match "^[^-\\]*\\*$([Regex]::Escape($Name))") {
         $OFS = ", "
         Write-Verbose "Commands is empty, but AllowedCommand ($AllowedCommand) contains $Name, so:"
         $Commands = @(Microsoft.PowerShell.Core\Get-Command $match)
       }
     }
     $cmd = $null
     if($Commands) {
       Write-Verbose "Commands $($Commands|% { $_.ModuleName + '\' + $_.Name })"
 
       if($Commands.Count -gt 1) {
         $cmd = @( $Commands | Where-Object { $_.Name -match "^$([Regex]::Escape($Name))" })[0]
       } else {
         $cmd = $Commands[0]
       }
     }
 
     if(!$cmd -and !$Search.Contains("-")) {
       $Commands = @(Microsoft.PowerShell.Core\Get-Command "Get-$Search" -ErrorAction SilentlyContinue -Module $AllowedModule | Where-Object { $_.Name -match "^Get-$([Regex]::Escape($Name))" })
       if($Commands) {
         if($Commands.Count -gt 1) {
           $cmd = @( $Commands | Where-Object { $_.Name -match "^$([Regex]::Escape($Name))" })[0]
         } else {
           $cmd = $Commands[0]
         }
       }
     }
 
     if(!$cmd -or $cmd.CommandType -eq "Alias") {
       if(($FullName = Get-Alias $Name -ErrorAction SilentlyContinue)) {
         if($FullName = $FullName.ResolvedCommand) {
           $cmd = Resolve-Command $FullName -AllowedModule $AllowedModule -AllowedCommand $AllowedCommand -ErrorAction SilentlyContinue
         }
       }
     }
     if(!$cmd) {
       if($PSBoundParameters.ContainsKey("AllowedModule")) {
         Write-Warning "No command '$Name' found in the allowed modules."
       } else {
         Write-Warning "No command '$Name' found in allowed modules. Expand-Alias defaults to only loaded modules, specify -AllowedModule `"*`" to allow ANY module"
       }     
       Write-Verbose "The current AllowedModules are: $($AllowedModule -join ', ')"
     }
     return $cmd
   }
 }
 
 function Protect-Script {
   #.Synopsis
   [CmdletBinding(ConfirmImpact="low",DefaultParameterSetName="Text")]
   param (
     [Parameter(Mandatory=$true, ParameterSetName="Text", Position=0)]
     [Alias("Text")]
     [string]$Script,
 
     [Parameter(Mandatory=$true)]
     [string[]]$AllowedModule,
 
     [Parameter()]
     [string[]]$AllowedCommand,
 
     [Parameter()]
     [string[]]$AllowedVariable
   )
 
   $Script = Expand-Alias -Script:$Script -AllowedModule:$AllowedModule -AllowedCommand $AllowedCommand -AllowedVariable $AllowedVariable
 
   if(![String]::IsNullOrWhiteSpace($Script)) {
 
     [string[]]$Commands = $AllowedCommand + (Microsoft.PowerShell.Core\Get-Command -Module:$AllowedModule | % { "{0}\{1}" -f $_.ModuleName, $_.Name})
     [string[]]$Variables = $AllowedVariable + (Microsoft.PowerShell.Core\Get-Module $AllowedModule | Select-Object -Expand ExportedVariables | Select-Object -Expand Keys)
 
     try {
       [ScriptBlock]::Create($Script).CheckRestrictedLanguage($Commands, $Variables, $false)
       return $Script
     } catch {
       $global:ProtectionError = $_
       Write-Warning $_
     }
   }
 
 }
 
 function Expand-Alias {
   #.Synopsis
   #.Description
   #.Example
    [CmdletBinding(ConfirmImpact="low",DefaultParameterSetName="Files")]
    param (
       [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="Files")]
       [Alias("FullName","PSChildName","PSPath")]
       [IO.FileInfo]$File,
 
       [Parameter(ParameterSetName="Files")] 
       [Switch]$InPlace,
 
       [Parameter(Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="Text")]
       [Alias("Text")]
       [string]$Script,
 
       [Parameter(Position=0, Mandatory=$false, ValueFromPipeline=$true, ParameterSetName="History")]
       [Alias("Id")]
       [Int[]]$History = (Get-History -Count 1).Id,
 
       [Parameter(Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="HistoryCount")]
       [Int]$Count,
 
       [string[]]$AllowedModule=$(@(Microsoft.PowerShell.Core\Get-Module | Select -Expand Name) + 'Microsoft.PowerShell.Core'),
 
       [Parameter()]
       [string[]]$AllowedCommand,
 
       [Parameter()]
       [string[]]$AllowedVariable,
 
       [Parameter()]
       [Switch]$Unqualified
    )
    begin {
       Write-Debug $PSCmdlet.ParameterSetName
    }
    process {
       
       switch( $PSCmdlet.ParameterSetName ) {
          "Files" {
             if($File -is [System.IO.FileInfo]){
                $Script = (Get-Content $File -Delim ([char]0))            
             }
          }
          "History" {
             $Script = (Get-History -Id $History | Select-Object -Expand CommandLine) -Join "`n"
          }
          "HistoryCount" {
             $Script = (Get-History -Count $Count | Select-Object -Expand CommandLine) -Join "`n"
          }
          "Text" {}
          default { throw "ParameterSet: $($PSCmdlet.ParameterSetName)" }
       }
 
       $ParseError = $null
       $Tokens = $null
       $AST = [System.Management.Automation.Language.Parser]::ParseInput($Script, [ref]$Tokens, [ref]$ParseError)
       $Global:Tokens = $Tokens
 
       if($ParseError) { 
          foreach($er in $ParseError) { Write-Error $er }
          throw "There was an error parsing script (See above). We cannot expand aliases until the script parses without errors."
       }
       $lastCommand = $Tokens.Count + 1
       :token for($t = $Tokens.Count -1; $t -ge 0; $t--) {
          Write-Verbose "Token $t of $($Tokens.Count)"
          $token = $Tokens[$t]
          switch -Regex ($token.Kind) {
             "Generic|Identifier" {
                 if($token.TokenFlags -eq 'CommandName') {
                    if($lastCommand -ne $t) {
                       $OFS = ", "
                       Write-Verbose "Resolve-Command -Name $Token.Text -AllowedModule $AllowedModule -AllowedCommand @($AllowedCommand)"
                       $Command = Resolve-Command -Name $Token.Text -AllowedModule $AllowedModule -AllowedCommand $AllowedCommand
                       if(!$Command) { return $null }
                    }
                    Write-Verbose "Unqualified? $Unqualified"
                    if(!$Unqualified -and $Command.ModuleName) { 
                       $CommandName = '{0}\{1}' -f $Command.ModuleName, $Command.Name
                    } else {
                       $CommandName = $Command.Name
                    }
                    $Script = $Script.Remove( $Token.Extent.StartOffset, ($Token.Extent.EndOffset - $Token.Extent.StartOffset)).Insert( $Token.Extent.StartOffset, $CommandName )
                 }
             }
             "Parameter" {
                Write-Verbose "lastCommand: $lastCommand ($t)"
                if($lastCommand -ge $t) {
                   for($c = $t; $c -ge 0; $c--) {
                     Write-Verbose "c: $($Tokens[$c].Text) ($($Tokens[$c].Kind) and $($Tokens[$c].TokenFlags))"
 
                      if(("Generic","Identifier" -contains $Tokens[$c].Kind) -and $Tokens[$c].TokenFlags -eq "CommandName" ) {
                         Write-Verbose "Resolve-Command -Name $($Tokens[$c].Text) -AllowedModule $AllowedModule -AllowedCommand $AllowedCommand"
                         $Command = Resolve-Command -Name $Tokens[$c].Text -AllowedModule $AllowedModule -AllowedCommand $AllowedCommand
                         if($Command) {
                           Write-Verbose "Command: $($Tokens[$c].Text) => $($Command.Name)"
                         }
 
                         $global:RCommand = $Command
                         if(!$Command) { return $null }
                         $lastCommand = $c
                         break
                      }
                   }
                }
             
                $short = "^" + $token.ParameterName
                $parameters = @($Command.ParameterSets | Select-Object -ExpandProperty Parameters | Where-Object {
                                 $_.Name -match $short -or $_.Aliases -match $short
                              } | Select-Object -Unique)
 
                Write-Verbose "Parameters: $($parameters | Select -Expand Name)"
                Write-Verbose "Parameters: $($Command.ParameterSets | Select-Object -ExpandProperty Parameters | Select -Expand Name) | ? Name -match $short"
                if($parameters.Count -ge 1) {
                   if($parameters[0].ParameterType -ne [System.Management.Automation.SwitchParameter]) {
                      if($Tokens.Count -ge $t -and ("Parameter","Semi","NewLine" -contains $Tokens[($t+1)].Kind)) {
                         Write-Warning "No value for parameter: $($parameters[0].Name), the next token is a $($Tokens[($t+1)].Kind) (Flags: $($Tokens[($t+1)].TokenFlags))"
                         $Script = ""
                         break token
                      }
                   }
                   $Script = $Script.Remove( $Token.Extent.StartOffset, ($Token.Extent.EndOffset - $Token.Extent.StartOffset)).Insert( $Token.Extent.StartOffset, "-$($parameters[0].Name)" )
                } else {
                   Write-Warning "Rejecting Non-Parameter: $($token.ParameterName)"
                   $Script = ""
                   break token
                }
                continue
             }
          }
       }
 
 
       if($InPlace) {
         if([String]::IsNullOrWhiteSpace($Script)) {
           Write-Warning "Script is empty after Expand-Alias, File ($File) not updated"
         } else {
           Set-Content -Path $File -Value $Script
         }
       } else {
         if([String]::IsNullOrWhiteSpace($Script)) {
           return
         } else {
           return $Script
         }
       }
    }
 }
 
 Set-Alias Resolve-Alias Expand-Alias
 Export-ModuleMember -Function * -Alias *
`

