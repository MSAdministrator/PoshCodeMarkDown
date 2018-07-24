---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3470
Published Date: 2012-06-20t21
Archived Date: 2012-06-30t20
---

# get-scope - 

## Description

a script to help you figure out what scope you are in “right now” (you have to always call it like

## Comments



## Usage



## TODO



## function

`get-scope`

## Code

`#
 #
 #.Synopsis 
 #.Parameter Invocation
 #.Parameter ToHost
 #.Parameter Foreground
 #.Parameter Background
 #.Example
 #
 #.Example
 #
 
 [CmdletBinding()]
 param(
    [Parameter(Position=0,Mandatory=$false)]
    [System.Management.Automation.InvocationInfo]$Invocation = $((Get-Variable -scope 1 'MyInvocation').Value)
 ,
    [Parameter(ParameterSetName="Debugging")]
    [Switch]$ToHost
 ,
    [Parameter(Position=1,ParameterSetName="Debugging")]
    [ConsoleColor]$Foreground = "Cyan"
 ,
    [Parameter(Position=2,ParameterSetName="Debugging")]
    [ConsoleColor]$Background = "Black"
 )
 end {
    function Get-ScopeDepth {
       trap { continue }
       $depth = 0
       do {
         Set-Variable scope_test -Value $depth -Scope ($depth++)
       } while($?)
       return $depth - 1
    }
    $depth = . Get-ScopeDepth
    $callstack = Get-PSCallStack
    
 	$IsInteractive = $true
    foreach ($entry in $callStack[1..($callStack.Count - 1)]) {
       write-host $($entry | fl | out-string) -fore Cyan
          continue
       }
       if (($entry.Command -eq 'prompt') -or ($entry.Location -eq 'prompt')) {
          write-host "Found the prompt, we're done here"
          break
       }
       $IsInteractive = $false
       break
    }
    
    $IsGlobal = 0 -eq (Get-Variable scope_test -Scope global).Value
 
    Remove-Variable scope_test
    Remove-Variable scope_test -Scope global -EA 0
 
    $PSScope = New-Object PSObject -Property @{ 
       Module = $Invocation.MyCommand.Module
       IsModuleScope = [bool]$Invocation.MyCommand.Module
       IsGlobalScope = $IsGlobal
       IsInteractive = $IsInteractive
       ScopeDepth  = $depth
       PipelinePosition = $Invocation.PipelinePosition
       PipelineLength = $Invocation.PipelineLength
       StackDepth = $callstack.Count - 1
       Invocation = $Invocation
       CallStack = $callstack
    }
    
    if($ToHost) {
       &{
          $PSScope, $PSScope.Invocation | Out-String 
       } | Write-Host -Foreground $Foreground -Background $Background
    } else {
       $PSScope
    }
 }
 #}
`

