---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1777
Published Date: 
Archived Date: 2010-04-17t07
---

# autoload (beta) - 

## Description

my first attempt at the autoload functionality of the korn shell

## Comments



## Usage



## TODO



## function

`skip-object`

## Code

`#
 #
 
 <#
 function Skip-Object {
 param( 
    [int]$First = 0, [int]$Last = 0, [int]$Every = 0, [int]$UpTo = 0,  
    [Parameter(Mandatory=$true,ValueFromPipeline=$true)]
    $InputObject
 )
 begin {
    if($Last) {
       $Queue = new-object System.Collections.Queue $Last
    }
    $e = $every; $UpTo++; $u = 0
 }
 process {
    $InputObject | where { --$First -lt 0 } | 
    foreach {
       if($Last) {
          $Queue.EnQueue($_)
          if($Queue.Count -gt $Last) { $Queue.DeQueue() }
       } else { $_ }
    } |
    foreach { 
       if(!$UpTo) { $_ } elseif( --$u -le 0) {  $_; $U = $UpTo }
    } |
    foreach { 
       if($every -and (--$e -le 0)) {  $e = $every  } else { $_ } 
    }
 }
 }
 #>
 <#
 autoload Skip-Object
 #>
 
 function autoload {
 [CmdletBinding()]
 param(
 [Parameter(Mandatory=$True,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
 [string[]]$Name
 )
 begin {
    $xlr8r = [type]::gettype("System.Management.Automation.TypeAccelerators")
    if(!$xlr8r::Get["PSParser"]) {
       $xlr8r::Add( "PSParser", "System.Management.Automation.PSParser, System.Management.Automation, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" )
    }
 
    function global:Autoloaded {
       [CmdletBinding()]Param()
       DYNAMICPARAM {
          Write-Verbose "Autoloaded DynamicParam: $($MyInvocation.InvocationName)"
          $paramDictionary = new-object System.Management.Automation.RuntimeDefinedParameterDictionary
          $externalScript = $ExecutionContext.InvokeCommand.GetCommand( $MyInvocation.InvocationName, [System.Management.Automation.CommandTypes]::ExternalScript )
          $parserrors = $null
          $prev = $null
          $script = $externalScript.ScriptContents
          [System.Management.Automation.PSToken[]]$tokens = [PSParser]::Tokenize( $script, [ref]$parserrors )
          [Array]::Reverse($tokens)
          
          ForEach($token in $tokens) {
             if($prev -and $token.Type -eq "Keyword" -and $token.Content -ieq "function" -and $prev.Content -eq $MyInvocation.InvocationName ) {
                $script = $script.Insert( $prev.Start, "global:" )
             }
             $prev = $token
          }
          Invoke-Expression $script | out-null
          $command = $ExecutionContext.InvokeCommand.GetCommand( $MyInvocation.InvocationName, [System.Management.Automation.CommandTypes]::Function )
          if(!$command) {
             throw "Something went wrong autoloading the $($MyInvocation.InvocationName) function. Function definition doesn't exist in script: $($externalScript.Path)"
          }
          foreach( $pkv in $command.Parameters.GetEnumerator() ){
             $parameter = $pkv.Value
             if( $parameter.Aliases -match "vb|db|ea|wa|ev|wv|ov|ob" ) { continue } 
             $param = new-object System.Management.Automation.RuntimeDefinedParameter( $parameter.Name, $parameter.ParameterType, $parameter.Attributes)
             $paramdictionary.Add($pkv.Key, $param)
          } 
          return $paramdictionary
 
       begin {
          Write-Verbose "Autoloaded Begin: $($MyInvocation.InvocationName)"
          Remove-Item Alias::$($MyInvocation.InvocationName)
          $command = $ExecutionContext.InvokeCommand.GetCommand( $MyInvocation.InvocationName, [System.Management.Automation.CommandTypes]::Function )
          $scriptCmd = {& $command @PSBoundParameters }
 
          $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
          $steppablePipeline.Begin($true)
       }
       process
       {
          Write-Verbose "Autoloaded Process: $($MyInvocation.InvocationName) ($_)"
          try {
             $steppablePipeline.Process($_)
          } catch {
             throw
          }
       }
 
       end
       {
          try {
             $steppablePipeline.End()
          } catch {
             throw
          }
          Write-Verbose "Autoloaded End: $($MyInvocation.InvocationName)"
       }
 
 }
 process {
    foreach($function in $Name) {
       Write-Verbose "Set-Alias $function Autoloaded -Scope global"
       Set-Alias $function Autoloaded -Scope global
    }
 }
 }
`

