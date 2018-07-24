---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 1261
Published Date: 
Archived Date: 2009-08-11t05
---

# wrapper 1 select-string - 

## Description

select-string has the funny behavior that it -encoding parameter could be default, but select-string uses unicode as default value. that means we do not find strings containing some western european letters like ‘����’ in ansi coded files. finally i want the choice default_or_oem, to search even my oem850 coded files.  here i fix only the default to ‘default’

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 [CmdletBinding()]
 param(
     [Parameter(ParameterSetName='Object', Mandatory=$true, ValueFromPipeline=$true)]
     [AllowEmptyString()]
     [AllowNull()]
     [System.Management.Automation.PSObject]
     ${InputObject},
 
     [Parameter(Mandatory=$true, Position=0)]
     [System.String[]]
     ${Pattern},
 
     [Parameter(ParameterSetName='File', Mandatory=$true, Position=1, ValueFromPipelineByPropertyName=$true)]
     [Alias('PSPath')]
     [System.String[]]
     ${Path},
 
     [Switch]
     ${SimpleMatch},
 
     [Switch]
     ${CaseSensitive},
 
     [Switch]
     ${Quiet},
 
     [Switch]
     ${List},
 
     [ValidateNotNullOrEmpty()]
     [System.String[]]
     ${Include},
 
     [ValidateNotNullOrEmpty()]
     [System.String[]]
     ${Exclude},
 
     [Switch]
     ${NotMatch},
 
     [Switch]
     ${AllMatches},
 
     [ValidateSet('unicode','utf7','utf8','utf32','ascii','bigendianunicode','default','oem')]
     [ValidateNotNullOrEmpty()]
     [System.String]
     ${Encoding},
 
     [ValidateNotNullOrEmpty()]
     [ValidateCount(1, 2)]
     [ValidateRange(0, 2147483647)]
     [System.Int32[]]
     ${Context})
 
 begin
 {
     try {
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
         {
             $PSBoundParameters['OutBuffer'] = 1
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Select-String', [System.Management.Automation.CommandTypes]::Cmdlet)
         $scriptCmd = {& $wrappedCmd @PSBoundParameters }
            
 
         #
         if (! $Encoding)
         {
             $PSBoundParameters.Encoding = 'default'
 
             <#
                 @(Select-String * -pattern Ärger -encoding default) +
                 @(Select-String * -pattern Ärger -encoding oem) |sort-object | Get-Unique
            #> 
           
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
 
 .ForwardHelpTargetName Select-String
 .ForwardHelpCategory Cmdlet
 
 #>
`

