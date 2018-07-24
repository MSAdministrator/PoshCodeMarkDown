---
Author: dmitry sotnikov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1606
Published Date: 2010-01-25t04
Archived Date: 2015-05-06t16
---

# select w/ subproperties - 

## Description

allows to use dots to specify object subproperties in select-object.

## Comments



## Usage



## TODO



## function

`select-object`

## Code

`#
 #
 <#
   This is a proxy function enhancing Select-Object by adding the
   ability to use subproperties in the Property parameters.
   
   For example:
   Set-Process | Select-Object ProcessName, StartTime.DayOfWeek
   
   This is the only changed behavior (properties with dots
   get evaluated as expressions) - everything else stays intact.
   
   For more information see:
 http://dmitrysotnikov.wordpress.com/2010/01/25/select-object-with-subproperties
 #>
 
 function Select-Object {
 [CmdletBinding(DefaultParameterSetName='DefaultParameter')]
 param(
   [Parameter(ValueFromPipeline=$true)]
   [System.Management.Automation.PSObject]
   ${InputObject},
 
   [Parameter(ParameterSetName='DefaultParameter', Position=0)]
   [System.Object[]]
   ${Property},
 
   [Parameter(ParameterSetName='DefaultParameter')]
   [System.String[]]
   ${ExcludeProperty},
 
   [Parameter(ParameterSetName='DefaultParameter')]
   [System.String]
   ${ExpandProperty},
 
   [Switch]
   ${Unique},
 
   [Parameter(ParameterSetName='DefaultParameter')]
   [ValidateRange(0, 2147483647)]
   [System.Int32]
   ${Last},
 
   [Parameter(ParameterSetName='DefaultParameter')]
   [ValidateRange(0, 2147483647)]
   [System.Int32]
   ${First},
 
   [Parameter(ParameterSetName='DefaultParameter')]
   [ValidateRange(0, 2147483647)]
   [System.Int32]
   ${Skip},
 
   [Parameter(ParameterSetName='IndexParameter')]
   [ValidateRange(0, 2147483647)]
   [System.Int32[]]
   ${Index})
 
 begin
 {
  try {
      $outBuffer = $null
      if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
      {
          $PSBoundParameters['OutBuffer'] = 1
      }
      
      if ($Property -ne $null) {
       $NewProperty = @()
       foreach ( $prop in $Property ) {
        if ($prop.GetType().Name -eq 'String') {
         if ($prop.Contains('.')) {
          [String] $exp = '$_.' + $prop
          $prop = @{Name=$prop; Expression = {Invoke-Expression ($exp)}}
         }
        }
        $NewProperty += $prop
       }
       $PSBoundParameters['Property'] = $NewProperty
      }
      
      $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Select-Object', 
          [System.Management.Automation.CommandTypes]::Cmdlet)
      $scriptCmd = {& $wrappedCmd @PSBoundParameters }
      $steppablePipeline = 
          $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
      $steppablePipeline.Begin($PSCmdlet)
  } catch {
      throw
  }
 }
 
 process
 {
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
 }
 <#
 
 .ForwardHelpTargetName Select-Object
 .ForwardHelpCategory Cmdlet
 
 #>
 }
`

