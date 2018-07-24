---
Author: jan egil ring
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2492
Published Date: 2011-02-07t07
Archived Date: 2016-06-01t11
---

# start-job proxy function - 

## Description

example on how to use proxy cmdlets in combination with object events.

## Comments



## Usage



## TODO



## function

`start-job`

## Code

`#
 #
 <#
 	Example on how to use Proxy Cmdlets in combination with object events.
 	For more information see:
 	
 	http://blog.powershell.no/2011/02/07/powershell-using-proxy-cmdlets-in-combination-with-object-events
 
 	For more information about proxy functions, see the following article on the
 	Microsoft PowerShell Team blog:
 
 	http://blogs.msdn.com/powershell/archive/2009/01/04/extending-and-or-modifing-commands-with-proxies.aspx
 #>
 
 function Start-Job {
 	<#
 		To create a proxy function for the Start-Job cmdlet, paste the results of the following command into the body of this function and then remove this comment:
 		[Management.Automation.ProxyCommand]::Create((New-Object Management.Automation.CommandMetaData (Get-Command Start-Job)))
 	#>
 
 [CmdletBinding(DefaultParameterSetName='ComputerName')]
 param(
     [Parameter(ValueFromPipelineByPropertyName=$true)]
     [System.String]
     ${Name},
 
     [Parameter(ParameterSetName='ComputerName', Mandatory=$true, Position=0)]
     [Alias('Command')]
     [System.Management.Automation.ScriptBlock]
     ${ScriptBlock},
 
     [System.Management.Automation.PSCredential]
     ${Credential},
 
     [Parameter(ParameterSetName='FilePathComputerName', Position=0)]
     [Alias('PSPath')]
     [System.String]
     ${FilePath},
 
     [System.Management.Automation.Runspaces.AuthenticationMechanism]
     ${Authentication},
 
     [Parameter(Position=1)]
     [System.Management.Automation.ScriptBlock]
     ${InitializationScript},
 
     [Switch]
     ${RunAs32},
 	
 	[System.Management.Automation.ScriptBlock]
     ${OnCompletionAction},
 
     [Parameter(ValueFromPipeline=$true)]
     [System.Management.Automation.PSObject]
     ${InputObject},
 
     [Alias('Args')]
     [System.Object[]]
     ${ArgumentList})
 
 begin
 {
     try {
         $outBuffer = $null
         if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
         {
             $PSBoundParameters['OutBuffer'] = 1
         }
         $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Start-Job', [System.Management.Automation.CommandTypes]::Cmdlet)
         
         $scriptCmdPipeline = ''
 
         if ($OnCompletionAction) {
             $PSBoundParameters.Remove('OnCompletionAction') | Out-Null
             $scriptCmdPipeline += " | foreach-object{
     `$job = Register-ObjectEvent -InputObject `$_ -EventName StateChanged -SourceIdentifier JobEndAlert -Action {
 	 if(`$sender.State -eq 'Completed')
      {
 	  `& {
 	  $OnCompletionAction
 	  }
       Unregister-Event -SourceIdentifier JobEndAlert -Force
      }
     }         
           }"
         }
 		
 		$scriptCmd = {& $wrappedCmd @PSBoundParameters }
 		
 		  $scriptCmd = $ExecutionContext.InvokeCommand.NewScriptBlock(
                 [string]$scriptCmd + $scriptCmdPipeline
             )
 		
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
 
 .ForwardHelpTargetName Start-Job
 .ForwardHelpCategory Cmdlet
 
 #>
 
 
 	
 }
`

