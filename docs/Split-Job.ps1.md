---
Author: arnoud jansveld
Publisher: 
Copyright: 
Email: 
Version: 0.8
Encoding: ascii
License: cc0
PoshCode ID: 2619
Published Date: 2012-04-20t09
Archived Date: 2015-03-07t05
---

# split-job - 

## Description

the split-job function provides easy multithreading at the command line or in a script. it was created from a system administrator’s point of view and is compatible with ps v1. supports importing functions, variables and snapins. for history and background please visit http

## Comments



## Usage



## TODO



## function

`test-webserver`

## Code

`#
 #
 ################################################################################
 ################################################################################
 
 function Split-Job {
 	param (
 		$Scriptblock = $(throw 'You must specify a command or script block!'),
 		[int]$MaxPipelines=10,
 		[switch]$UseProfile,
 		[string[]]$Variable,
 		[string[]]$Function = @(),
 		[string[]]$Alias = @(),
 		[string[]]$SnapIn
 	) 
 	
 	function Init ($InputQueue){
 		$Queue = [Collections.Queue]::Synchronized([Collections.Queue]@($InputQueue))
 		$QueueLength = $Queue.Count
 		if ($MaxPipelines -gt $QueueLength) {$MaxPipelines = $QueueLength}
 		$Script  = "Set-Location '$PWD'; "
 		$Script += {
 			$SplitJobQueue = $($Input)
 			& {
 				trap {continue}
 				while ($SplitJobQueue.Count) {$SplitJobQueue.Dequeue()}
 			} |
 		}.ToString() + $Scriptblock
 	
 		$Pipelines = New-Object System.Collections.ArrayList
 		
 		$ImportItems = ($Function -replace '^','Function:') + 
 			($Alias -replace '^','Alias:') |
 			Get-Item | select PSPath, Definition
 		$stopwatch = New-Object System.Diagnostics.Stopwatch
 		$stopwatch.Start()
 	}
 
     function Add-Pipeline {
         $Runspace = [System.Management.Automation.Runspaces.RunspaceFactory]::CreateRunspace($Host)
         $Runspace.Open()
 		if (!$?) {throw "Could not open runspace!"}
 		$Runspace.SessionStateProxy.SetVariable('SplitJobRunSpace', $True)
 		
 		function CreatePipeline {
 			param ($Data, $Scriptblock)
 			$Pipeline = $Runspace.CreatePipeline($Scriptblock)
 			if ($Data) {
 				$Null = $Pipeline.Input.Write($Data, $True)
 				$Pipeline.Input.Close()
 			}
 			$Null = $Pipeline.Invoke()
             $Pipeline.Dispose()
 		}
 		
         if ($UseProfile) {
             CreatePipeline -Script "`$PROFILE = '$PROFILE'; . `$PROFILE"
         }
 		if ($Variable) {
             foreach ($var in (Get-Variable $Variable -Scope 2)) {
                 trap {continue}
                 $Runspace.SessionStateProxy.SetVariable($var.Name, $var.Value)
             }
         }
         if ($ImportItems) {
 			CreatePipeline $ImportItems {
 				foreach ($item in $Input) {New-Item -Path $item.PSPath -Value $item.Definition}
 			}
         }
 		if ($SnapIn) {
 			CreatePipeline (Get-PSSnapin $Snapin -Registered) {$Input | Add-PSSnapin}
 		}
         $Pipeline = $Runspace.CreatePipeline($Script)
         $Null = $Pipeline.Input.Write($Queue)
         $Pipeline.Input.Close()
         $Pipeline.InvokeAsync()
         $Null = $Pipelines.Add($Pipeline)
     }
 
     function Remove-Pipeline ($Pipeline) {
         $Runspace = $Pipeline.RunSpace
         $Pipeline.Dispose()
         $Runspace.Close()
         $Runspace.Dispose()
         $Pipelines.Remove($Pipeline)
     }
 
 	. Init $Input
     while ($Pipelines.Count -lt $MaxPipelines -and $Queue.Count) {Add-Pipeline} 
 
     while ($Pipelines.Count) {
 		if (($stopwatch.ElapsedMilliseconds - $LastUpdate) -gt 1000) {
 			$Completed = $QueueLength - $Queue.Count - $Pipelines.count
 			$LastUpdate = $stopwatch.ElapsedMilliseconds
 			$SecondsRemaining = $(if ($Completed) {
 				(($Queue.Count + $Pipelines.Count)*$LastUpdate/1000/$Completed)
 			} else {-1})
     	    Write-Progress 'Split-Job' ("Queues: $($Pipelines.Count)  Total: $($QueueLength)  " +
 			"Completed: $Completed  Pending: $($Queue.Count)")  `
             -PercentComplete ([Math]::Max((100 - [Int]($Queue.Count + $Pipelines.Count)/$QueueLength*100),0)) `
 			-CurrentOperation "Next item: $(trap {continue}; if ($Queue.Count) {$Queue.Peek()})" `
 			-SecondsRemaining $SecondsRemaining
 		}
         foreach ($Pipeline in @($Pipelines)) {
             if ( -not $Pipeline.Output.EndOfPipeline -or -not $Pipeline.Error.EndOfPipeline ) {
                 $Pipeline.Output.NonBlockingRead()
                 $Pipeline.Error.NonBlockingRead() | Out-Default
             } else {
                 if ($Pipeline.PipelineStateInfo.State -eq 'Failed') {
                     $Pipeline.PipelineStateInfo.Reason.ErrorRecord | 
 						Add-Member NoteProperty writeErrorStream $True -PassThru |
 							Out-Default
                     if ($Queue.Count -lt $QueueLength) {Add-Pipeline}
                 }
                 Remove-Pipeline $Pipeline
             }
         }
         Start-Sleep -Milliseconds 100
     }
 }
`

