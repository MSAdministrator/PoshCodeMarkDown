---
Author: robert riegler
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1209
Published Date: 
Archived Date: 2009-07-21t09
---

# format-tableplus - 

## Description

robert,  the problem is that this script requires version 2.0 … that “getsteppablepipeline” is a new feature. i forgot to mark it.

## Comments



## Usage



## TODO



## function

`format-tableplus`

## Code

`#
 #
 function Format-TablePlus() {
 [CmdletBinding()]
 param(
    [Switch]
    ${AutoSize},
 
    [Switch]
    ${HideTableHeaders},
 
    [Switch]
    ${Wrap},
 
    [Parameter(Position=0)]
    [System.Object[]]
    ${Property},
 
    [System.Object]
    ${GroupBy},
 
    [System.String]
    ${View},
 
    [Switch]
    ${ShowError},
 
    [Switch]
    ${DisplayError},
 
    [Switch]
    ${Force},
 
    [ValidateSet('CoreOnly','EnumOnly','Both')]
    [System.String]
    ${Expand},
 
    [System.Int32]
    ${Width} = $Host.Ui.RawUI.BufferSize.Width,
 
    [Switch]
    ${PadEnd},
 
    [Parameter(ValueFromPipeline=$true)]
    [System.Management.Automation.PSObject]
    ${InputObject}
 )
 
 begin
 {
    try {
       $outBuffer = $null
       if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
       {
          $PSBoundParameters['OutBuffer'] = 1
       }
       $null = $PSBoundParameters.Remove("Width")
       $null = $PSBoundParameters.Remove("TrimEnd")
       
       $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Format-Table', [System.Management.Automation.CommandTypes]::Cmdlet)
       
       $scriptCmd = &{ 
          if($PadEnd) {
             {& $wrappedCmd @PSBoundParameters | Out-String -Stream -Width $Width }
          } else {
             {& $wrappedCmd @PSBoundParameters | Out-String -Stream -Width $Width | % { $_.TrimEnd() } }
          }
       }
       $steppablePipeline = $scriptCmd.($myInvocation.CommandOrigin)
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
 
 .ForwardHelpTargetName Format-Table
 .ForwardHelpCategory Cmdlet
 
 #>
 }
`

