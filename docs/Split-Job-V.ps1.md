---
Author: stephen mills
Publisher: 
Copyright: 
Email: 
Version: 1.2.1
Encoding: ascii
License: cc0
PoshCode ID: 3679
Published Date: 2012-10-04t13
Archived Date: 2016-12-07t04
---

# split-job v - 

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
 <#
 ################################################################################
 ##
 ##
 ##
 ##
 ################################################################################
 #>
 
 function Split-Job
 {
 	param (
 		[Parameter(Position=0, Mandatory=$true)]$Scriptblock,
 		[Parameter()][int]$MaxPipelines=10,
 		[Parameter()][switch]$UseProfile,
 		[Parameter()][string[]]$Variable,
 		[Parameter()][string[]]$Function = @(),
 		[Parameter()][string[]]$Alias = @(),
 		[Parameter()][string[]]$SnapIn,
 		[Parameter()][float]$MaxDuration = $( [Int]::MaxValue ),
 		[Parameter()][string]$ProgressInfo ='',
 		[Parameter()][int]$ProgressID = 0,
 		[Parameter()][switch]$NoProgress,
 		[Parameter()][int]$DisplayInterval = 300,
 		[Parameter()][scriptblock]$InitializeScript,
 		[Parameter(ValueFromPipeline=$true)][object[]]$InputObject
 	)
 
 	begin
 	{
 		$StartTime = Get-Date
 		#$DisplayTime = $StartTime.AddMilliseconds( - $DisplayInterval )
 		$ExitForced = $false
 
 
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
 				} }.ToString() + '|' + $Scriptblock
 
 			$Pipelines = New-Object System.Collections.ArrayList
 
 			$ImportItems = ($Function -replace '^','Function:') +
 				($Alias -replace '^','Alias:') |
 				Get-Item | select PSPath, Definition
 			$stopwatch = [System.Diagnostics.Stopwatch]::StartNew()
 		}
 
 		function Add-Pipeline {
 			$Runspace = [System.Management.Automation.Runspaces.RunspaceFactory]::CreateRunspace($Host)
 			$Runspace.Open()
 			if (!$?) { throw "Could not open runspace!" }
 			$Runspace.SessionStateProxy.SetVariable('SplitJobRunSpace', $True)
 
 			function CreatePipeline
 			{
 				param ($Data, $Scriptblock)
 				$Pipeline = $Runspace.CreatePipeline($Scriptblock)
 				if ($Data)
 				{
 					$Null = $Pipeline.Input.Write($Data, $True)
 					$Pipeline.Input.Close()
 				}
 				$Null = $Pipeline.Invoke()
 				$Pipeline.Dispose()
 			}
 
 			
 			if ($UseProfile)
 			{
 				CreatePipeline -Script "`$PROFILE = '$PROFILE'; . `$PROFILE"
 			}
 
 			if ($Variable)
 			{
 				foreach ($var in (Get-Variable $Variable -Scope 2))
 				{
 					trap {continue}
 					$Runspace.SessionStateProxy.SetVariable($var.Name, $var.Value)
 				}
 			}
 			if ($ImportItems)
 			{
 				CreatePipeline $ImportItems {
 					foreach ($item in $Input) {New-Item -Path $item.PSPath -Value $item.Definition}
 				}
 			}
 			if ($SnapIn)
 			{
 				CreatePipeline (Get-PSSnapin $Snapin -Registered) {$Input | Add-PSSnapin}
 			}
 			
 			if ($InitializeScript -ne $null)
 			{
 				CreatePipeline -Scriptblock $InitializeScript
 			}
 
 			$Pipeline = $Runspace.CreatePipeline($Script)
 			$Null = $Pipeline.Input.Write($Queue)
 			$Pipeline.Input.Close()
 			$Pipeline.InvokeAsync()
 			$Null = $Pipelines.Add($Pipeline)
 		}
 
 		function Remove-Pipeline ($Pipeline)
 		{
 			$Pipeline.RunSpace.CloseAsync()
 			#$Pipeline.Dispose()
 			$Pipelines.Remove($Pipeline)
 		}
 	}
 
 	end
 	{
 		
 
 
 		. Init $Input
 		try
 		{
 			while ($Pipelines.Count -lt $MaxPipelines -and $Queue.Count) {Add-Pipeline}
 
 			while ($Pipelines.Count)
 			{
 				if (-not $NoProgress -and $Stopwatch.ElapsedMilliseconds -ge $DisplayInterval)
 				{
 					$Completed = $QueueLength - $Queue.Count - $Pipelines.count
 					$Stopwatch.Reset()
 					$Stopwatch.Start()
 					#$LastUpdate = $stopwatch.ElapsedMilliseconds
 					$PercentComplete = (100 - [Int]($Queue.Count)/$QueueLength*100)
 					$Duration = (Get-Date) - $StartTime
 					$DurationString = [timespan]::FromSeconds( [Math]::Floor($Duration.TotalSeconds)).ToString()
 					$ItemsPerSecond = $Completed / $Duration.TotalSeconds
 					$SecondsRemaining = [math]::Round(($QueueLength - $Completed)/ ( .{ if ($ItemsPerSecond -eq 0 ) { 0.001 } else { $ItemsPerSecond}}))
 					
 					Write-Progress -Activity "** Split-Job **  *Press Esc to exit*  Next item: $(trap {continue}; if ($Queue.Count) {$Queue.Peek()})" `
 						-status "Queues: $($Pipelines.Count) QueueLength: $($QueueLength) StartTime: $($StartTime)  $($ProgressInfo)" `
 						-currentOperation  "$( . { if ($ExitForced) { 'Aborting Job!   ' }})Completed: $($Completed) Pending: $($QueueLength- ($QueueLength-($Queue.Count + $Pipelines.Count))) RunTime: $($DurationString) ItemsPerSecond: $([math]::round($ItemsPerSecond, 3))"`
 						-PercentComplete $PercentComplete `
 						-Id $ProgressID `
 						-SecondsRemaining $SecondsRemaining
 				}	
 				foreach ($Pipeline in @($Pipelines))
 				{
 					if ( -not $Pipeline.Output.EndOfPipeline -or -not $Pipeline.Error.EndOfPipeline)
 					{
 						$Pipeline.Output.NonBlockingRead()
 						$Pipeline.Error.NonBlockingRead() | % { Write-Error -ErrorRecord $_ }
 
 					} else
 					{
 						if ($Pipeline.PipelineStateInfo.State -eq 'Failed')
 						{
 							Write-Error $Pipeline.PipelineStateInfo.Reason
 							
 							if ($Queue.Count -lt $QueueLength) {Add-Pipeline}
 						}
 						Remove-Pipeline $Pipeline
 					}
 					if ( ((Get-Date) - $StartTime).TotalSeconds -ge $MaxDuration -and -not $ExitForced)
 					{
 						Write-Warning "Aborting job! The MaxDuration of $MaxDuration seconds has been reached. Inputs that have not been processed will be skipped."
 						$ExitForced=$true
 					}
 					
 					if ($ExitForced) { $Pipeline.StopAsync(); Remove-Pipeline $Pipeline }
 				}
 				while ($Host.UI.RawUI.KeyAvailable)
 				{
 					if ($Host.ui.RawUI.ReadKey('NoEcho,IncludeKeyDown,IncludeKeyUp').VirtualKeyCode -eq 27 -and !$ExitForced)
 					{
 						$Queue.Clear();
 						Write-Warning 'Aborting job! Escape pressed! Inputs that have not been processed will be skipped.'
 						$ExitForced = $true;
 						#{
 						#}
 					}		
 				}
 				if ($Pipelines.Count) {Start-Sleep -Milliseconds 50}
 			}
 
 			Write-Progress -Completed -Activity "`0" -Status "`0"
 
 			[GC]::Collect()
 		}
 		finally
 		{
 			foreach ($Pipeline in @($Pipelines))
 			{
 				if ( -not $Pipeline.Output.EndOfPipeline -or -not $Pipeline.Error.EndOfPipeline)
 				{
 					Write-Warning 'Pipeline still runinng.  Stopping Async.'
 					$Pipeline.StopAsync()
 					Remove-Pipeline $Pipeline
 				}
 			}
 		}
 	}
 }
`

