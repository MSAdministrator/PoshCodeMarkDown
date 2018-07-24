---
Author: bielawb
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 4151
Published Date: 2014-05-10t22
Archived Date: 2017-03-12t05
---

# astsplatting - 

## Description

another fun script that used powershell v3 asts. have fun!

## Comments



## Usage



## TODO



## function

`invoke-static`

## Code

`#
 #
 
 function Invoke-Static {
 <#
     .Synopsis
         Invokes static methods with pseudo-splatting.
 
     .Description
         This function is using Abstract Syntax Tree to parse code passed.
         Any [YourType]::StaticMethod($Arguments) call will be inspected.
         If $Arguments can be used as hashtable - function will attempt to 'splat' it.
 
     .Example
         $Power = @{ x = 2; y = 3 }
         Invoke-Static -ScriptBlock { [math]::Pow($Power) }
         8
 
         Keys of passed hashtable will be matched agains overloads for System.Math static method Pow.
         Because keys match arguments (x, y) - function work as expected
 
     .Example 
         $Param = @{ input = 'This-Is-Only-Test'; pattern = '-' }
         Invoke-Static '[regex]::Split($Param)'
         This
         Is
         Only
         Test
 
         Using parameter set with expression you can simply pass whole expression as a string.
     
     .Example
         $CheckFeb = @{ year = 2010; month = 2 }
         ^ [DateTime]::DaysInMonth($CheckFeb)
         28
 
         Same syntax without quoting will cause $CheckFeb to be another positional parameter.
         This syntax is possible only if defaul function alias (^) is defined.
 #>
 
 [CmdletBinding(
     DefaultParameterSetName = 'script'
 )]
 param (
     
     [Parameter(
         ParameterSetName = 'script',
         Position = 0,
         Mandatory
     )]
     [scriptblock]$ScriptBlock,
 
     [Parameter(
         ParameterSetName = 'string',
         Position = 0,
         Mandatory
     )]
     [string]$Expression,
 
     [Parameter(
         ParameterSetName = 'string',
         Position = 1
     )]$Splatted
 )
     if ($Expression) {
         if ($Splatted) {
             $Expression = "{0}({1})" -f $Expression, '$Splatted'
         }
         $ScriptBlock = [scriptblock]::Create($Expression)
     }
 
     foreach ($Invoke in $ScriptBlock.Ast.FindAll({
         $args[0] -is [Management.Automation.Language.InvokeMemberExpressionAst] -and
         $args[0].Static -and 
         $args[0].Arguments.Count -eq 1 -and
         $args[0].Expression -is [Management.Automation.Language.TypeExpressionAst]
     }, $false)) {
         try {
             $Type = [type]$Invoke.Expression.TypeName.FullName
             $Method = $Invoke.Member.Value
             $Name = $Invoke.Arguments.VariablePath.UserPath -replace '\w+:'
             $Argument = Get-Variable -Name $Name -ValueOnly
             if ($Hash = $Argument -as [hashtable]) {
                 $Selected = $Type.GetMethods() |
                     where { 
                         $_.Name -eq $Method -and (
                         $_.GetParameters().Name.Count -eq
                         $Hash.Keys.Count ) -and (
                         $Hash.Keys.Count -eq   
                         ($_.GetParameters() | 
                             where { $_.Name -in $Hash.Keys }).Count
                         )
                     }
                 foreach ($Item in $Selected) {
                     $ParamList = New-Object System.Collections.ArrayList
                     $Item.GetParameters() | sort Position |
                     ForEach-Object {
                         $ParamList.Add($Argument[$_.Name] -as $_.ParameterType) | Out-Null
                     }
                     $Item.Invoke($Item,$ParamList.ToArray())
                 }
             } else {
                 $Type::$Method.Invoke($Argument)
             }
         } catch {
             Write-Verbose "We don't care about errors. If you do, it was: $_"
         }    
     }
 }
 
 New-Alias -Name ^ -Value Invoke-Static -Force
 
 
 $ToSplat = @{
     format = "Using 'splatting' hashtable: {0:N2} and {1:N3}"
     args = [math]::PI, [math]::E
 }
 
 $Array = @(
     "Using 'splatting' array: {0:N3}"
     [math]::PI
 )
 
 ^ { 
     $Dont = @{
         Define = 'Anything'
         Inside = 'Block'
     }
 
     [string]::Format($ToSplat) 
     [string]::Format($Array)
     $This::Silently($Fail)
     
     'Everything else is simply ignored'
     Stop-Computer -Force
 }
 
 $Replacer = @{
     input = 'bartosz bielawski'
     pattern = '\b\w'
     evaluator = { $args[0].Value.ToUpper() }
 }
 ^ { [regex]::Replace($Replacer) }
 
 $Replacer.evaluator = { $args[0].Value * 5 }
 ^ { [regex]::Replace($Replacer) }
 
 $Replacer.evaluator = { "[{0}]" -f $args[0].Value.ToUpper() }
 ^ { [regex]::Replace($Replacer) }
 
`

