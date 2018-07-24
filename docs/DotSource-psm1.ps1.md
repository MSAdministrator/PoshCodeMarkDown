---
Author: bielawb
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4125
Published Date: 2013-04-22t11
Archived Date: 2013-05-02t09
---

# dotsource.psm1 - 

## Description

this is very simple module that you can use with powershell v3. it’s using powershell abstract syntax tree to import functions from any script without running any code.

## Comments



## Usage



## TODO



## function

`import-script`

## Code

`#
 #
 $accelerators = [psobject].Assembly.GetType(
     'System.Management.Automation.TypeAccelerators'
 )
 
 foreach ($Type in 
     'Parser', 
     'FunctionDefinitionAst',
     'AssignmentStatementAst',
     'VariableExpressionAst',
     'TokenKind') 
 {
     $accelerators::Add(
         $Type,
         "System.Management.Automation.Language.$Type"
     )    
 }
 
 
 function Import-Script {
 param (
     $Path,
     [ValidateSet(
         'Script',
         'Global',
         'Local'
     )]
     $Scope = 'Global',
     [switch]$IncludeVariables
 )
 
     $fullName = (Resolve-Path $Path).ProviderPath
     $ast = [Parser]::ParseFile(
         $fullName,
         [ref]$null,
         [ref]$null
     )
     foreach ($function in $ast.FindAll({
         $args[0] -is [FunctionDefinitionAst]
         }, $false)) {
         $define = $false
         switch -Regex ($function.Name) {
             '^(?!.*:)' {
                 $define = $true
                 $name = $function.Name
                 break
             }
             '(Global|Script|Local):' {
                 $define = $true
                 $name = $function.Name -replace '.*:'
             }
             default {
                 $define = $false
             }
         }
         if ($define) {
             & ([scriptblock]::Create("
                 function $Scope`:$name
                 $($function.Body)
                 "
             ))
         }
     }
     if ($IncludeVariables) {
     foreach ($variable in $ast.FindAll({
         $args[0] -is [AssignmentStatementAst] -and
         $args[0].Operator -eq [TokenKind]::Equals -and
         $args[0].Left -is [VariableExpressionAst]
         }, $false)) {
         $define = $false
         switch -Regex ($variable.Left.VariablePath) {
             '^(?!.*:)' {
                 $define = $true
                 $name = $variable.Left.VariablePath
                 break
             }
             '(Global|Script|Local):' {
                 $define = $true
                 $name = $variable.Left.VariablePath -replace '.*:'
             }
             default {
                 $define = $false
             }
         }
         if ($define) {
             & ([scriptblock]::Create(
                 "`$$Scope`:$name = $(
                 $variable.Right.Extent.Text
                 )"
             ))
         }
     }
     }
 }
 
 New-Alias -Name .. -Value Import-Script -Force
 Export-ModuleMember -Function * -Alias *
`

