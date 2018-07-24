---
Author: dmitry sotnikov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1590
Published Date: 2010-01-19t05
Archived Date: 2017-05-16t03
---

# export-csv -append - 

## Description

add this function to your profile to make export-csv cmdlet handle -append parameter

## Comments



## Usage



## TODO



## function

`export-csv`

## Code

`#
 #
 
 <#
   This Export-CSV behaves exactly like native Export-CSV
   However it has one optional switch -Append
   Which lets you append new data to existing CSV file: e.g.
   Get-Process | Select ProcessName, CPU | Export-CSV processes.csv -Append
   
   For details, see
   http://dmitrysotnikov.wordpress.com/2010/01/19/export-csv-append/
   
   (c) Dmitry Sotnikov
 #>
 
 function Export-CSV {
 [CmdletBinding(DefaultParameterSetName='Delimiter',
   SupportsShouldProcess=$true, ConfirmImpact='Medium')]
 param(
  [Parameter(Mandatory=$true, ValueFromPipeline=$true,
            ValueFromPipelineByPropertyName=$true)]
  [System.Management.Automation.PSObject]
  ${InputObject},
 
  [Parameter(Mandatory=$true, Position=0)]
  [Alias('PSPath')]
  [System.String]
  ${Path},
  
  [Switch]
  ${Append},
 
  [Switch]
  ${Force},
 
  [Switch]
  ${NoClobber},
 
  [ValidateSet('Unicode','UTF7','UTF8','ASCII','UTF32','BigEndianUnicode','Default','OEM')]
  [System.String]
  ${Encoding},
 
  [Parameter(ParameterSetName='Delimiter', Position=1)]
  [ValidateNotNull()]
  [System.Char]
  ${Delimiter},
 
  [Parameter(ParameterSetName='UseCulture')]
  [Switch]
  ${UseCulture},
 
  [Alias('NTI')]
  [Switch]
  ${NoTypeInformation})
 
 begin
 {
  $AppendMode = $false
  
  try {
   $outBuffer = $null
   if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
   {
       $PSBoundParameters['OutBuffer'] = 1
   }
   $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Export-Csv',
     [System.Management.Automation.CommandTypes]::Cmdlet)
         
         
 	$scriptCmdPipeline = ''
 
 	if ($Append) {
   
 		$PSBoundParameters.Remove('Append') | Out-Null
     
   if ($Path) {
    if (Test-Path $Path) {        
     $AppendMode = $true
     
     if ($Encoding.Length -eq 0) {
      $Encoding = 'ASCII'
     }
     
     $scriptCmdPipeline += 'ConvertTo-Csv -NoTypeInformation '
     
     if ( $UseCulture ) {
      $scriptCmdPipeline += ' -UseCulture '
     }
     if ( $Delimiter ) {
      $scriptCmdPipeline += " -Delimiter '$Delimiter' "
     } 
     
     $scriptCmdPipeline += ' | Foreach-Object {$start=$true}'
     $scriptCmdPipeline += '{if ($start) {$start=$false} else {$_}} '
     
     $scriptCmdPipeline += " | Out-File -FilePath '$Path' -Encoding '$Encoding' -Append "
     
     if ($Force) {
      $scriptCmdPipeline += ' -Force'
     }
 
     if ($NoClobber) {
      $scriptCmdPipeline += ' -NoClobber'
     }   
    }
   }
  } 
   
 
   
  $scriptCmd = {& $wrappedCmd @PSBoundParameters }
  
  if ( $AppendMode ) {
   $scriptCmd = $ExecutionContext.InvokeCommand.NewScriptBlock(
       $scriptCmdPipeline
     )
  } else {
   $scriptCmd = $ExecutionContext.InvokeCommand.NewScriptBlock(
       [string]$scriptCmd
     )
  }
 
  $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
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
 
 .ForwardHelpTargetName Export-Csv
 .ForwardHelpCategory Cmdlet
 
 #>
 
 }
`

