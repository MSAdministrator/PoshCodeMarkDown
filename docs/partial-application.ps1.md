---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1687
Published Date: 2010-03-10t16
Archived Date: 2016-11-14t09
---

# partial application - 

## Description

a proof of concept module implementing partial application (not currying) of functions and cmdlets in powershell. this is a functional language technique often used in languages like haskell, ml etc.

## Comments



## Usage



## TODO



## function

`get-parameterdictionary`

## Code

`#
 #
 Set-StrictMode -Version 2
 
 $commonParameters = @("Verbose",
                       "Debug",
                       "ErrorAction",
                       "WarningAction",
                       "ErrorVariable",
                       "WarningVariable",
                       "OutVariable",
                       "OutBuffer")
 
 <#
 .SYNOPSIS
     Support function for partially-applied cmdlets and functions.
 #>
 function Get-ParameterDictionary {
     [outputtype([Management.Automation.RuntimeDefinedParameterDictionary])]
     [cmdletbinding()]
     param(
         [validatenotnull()]
         [management.automation.commandinfo]$CommandInfo,
         [validatenotnull()]
         [management.automation.pscmdlet]$PSCmdletContext = $PSCmdlet
     )
     
     $rdpd = new-object Management.Automation.RuntimeDefinedParameterDictionary
 
     try {
         if ($CommandInfo.parametersets.count > 1) {
             $parameters = $CommandInfo.ParameterSets[[string]$CommandInfo.DefaultParameterSet].parameters
         } else {
             $parameters = $CommandInfo.parameters.getenumerator() | % {$CommandInfo.parameters[$_.key]}
         }        
                 
         $parameters | % {
             
             write-verbose "testing $($_.name)"
                                     
             if ($commonParameters -like $_.Name) {                                  
                 
                 write-verbose "skipping common parameter $($_.name)"
                 
             } else {
                 
                 $rdp = new-object management.automation.runtimedefinedparameter
                 $rdp.Name = $_.Name
                 $rdp.ParameterType = $_.ParameterType
                 
                 $pa = new-object system.management.automation.parameterattribute
                 $pa.ParameterSetName = $PSCmdletContext.ParameterSetName
                 $rdp.Attributes.Add($pa)
                 
                 $rdpd.add($_.Name, $rdp)
             }
             
         }
     } catch {
     
         Write-Warning "Error getting parameter dictionary: $_"
     }
     
     $rdpd
 }
 
 <#
 .SYNOPSIS
     Function that accepts a FunctionInfo or CmdletInfo reference and one or more parameters
     and returns a FunctionInfo bound to those parameter(s) and their value(s.)
 .DESCRIPTION
     Function that accepts a FunctionInfo or CmdletInfo reference and one or more parameters
     and returns a FunctionInfo bound to those parameter(s) and their value(s.)
     
     Any parameters "merged" into the function are removed from the available parameters for
     future invocations. Multiple chained merge-parameter calls are permitted.
 .EXAMPLE
 
     First, we define a simple function:
     
     function test {
         param($a, $b, $c, $d);
         "a: $a; b: $b; c:$c; d:$d"
     }
     
     Now we merge -b parameter into functioninfo with the static value of 5, returning a new
     functioninfo:
     
     ps> $x = merge-parameter (gcm test) -b 5
     
     We execute the new functioninfo with the & (call) operator, passing in the remaining 
     arguments:
     
     ps> & $x -a 2 -c 4 -d 9
     a: 2; b: 5; c: 4; d: 9
     
     Now we merge two new parameters in, -c with the value 3 and -d with 5:
     
     ps> $y = merge-parameter $x -c 3 -d 5
     
     Again we call $y with the remaining named parameter -a:
     
     ps> & $y -a 2
     a: 2; b: 5; c: 3; d: 5
 .EXAMPLE
 
     Cmdlets can also be subject to partial application. In this case we create a new
     function with the returned functioninfo:
     
     ps> si function:get-function (merge-parameter (gcm get-command) -commandtype function)
     ps> get-function
     <lists all commands of commandtype "function">            
 .PARAMETER _CommandInfo
     The FunctionInfo or CmdletInfo into which to merge (apply) parameter(s.)
     
     The parameter is named with a leading underscore character to prevent parameter
     collisions when exposing the targetted command's parameters and dynamic parameters.
 .INPUTS
     FunctionInfo or CmdletInfo
 .OUTPUTS
     FunctionInfo
 #>
 function Merge-Parameter {    
     [OutputType([Management.Automation.FunctionInfo])]
     [CmdletBinding()]
     param(
         [parameter(position=0, mandatory=$true)]
         [validatenotnull()]
         [validatescript({
             ($_ -is [management.automation.functioninfo]) -or `
             ($_ -is [management.automation.cmdletinfo])
         })]
         [management.automation.commandinfo]$_Command
     )
     
     dynamicparam {
         if ((test-path variable:_command)) {
             Get-ParameterDictionary $_Command $PSCmdlet
         }
     }
 
     begin {
         write-verbose "merge-parameter: begin"
         
         $mergedParameters = new-object 'collections.generic.dictionary[string,object]' $PSBoundParameters
         
         $mergedParameters.remove("_Command") > $null
         
         $commonParameters | % {
             if ($mergedParameters.ContainsKey($_)) {
                 $mergedParameters.Remove($_)  > $null
             }
         }
     }
     
     process {
         write-verbose "merge-parameter: process"
         
         $temp = [guid]::NewGuid()
 
         $target = $_Command
 
         $partial = {
             [cmdletbinding()]
             param()
             
             dynamicparam
             {                
                 $targetRdpd = Get-ParameterDictionary $target $PSCmdlet
         
                 $mergedParameters.keys | % {
                     $targetRdpd.remove($_) > $null
                 }
                 $targetRdpd
             }
             begin {
                 write-verbose "i have $($mergedParameters.count) fixed parameter(s)."
                 write-verbose "i have $($targetrdpd.count) remaining parameter(s)"
             }
             process {
                 $boundParameters = $PSCmdlet.MyInvocation.BoundParameters
                 
                 $commonParameters | % {
                     if ($boundParameters.ContainsKey($_)) {
                         $boundParameters.Remove($_)  > $null
                     }
                 }
                 
                 . $target @mergedParameters @boundParameters
                 
                 if ($args) {
                     write-warning "received $($args.count) arg(s) not part of function."
                 }
             }
         }
         
         new-item -Path function:$temp -Value $partial.GetNewClosure()
     }
     
     end {
         rm function:$temp
     }    
 }
 
 new-alias papp Merge-Parameter -force
 
 Export-ModuleMember -Alias papp -Function Merge-Parameter, Get-ParameterDictionary
`

