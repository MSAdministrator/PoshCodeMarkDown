---
Author: foureight84
Publisher: 
Copyright: 
Email: 
Version: 0.2
Encoding: ascii
License: cc0
PoshCode ID: 2315
Published Date: 
Archived Date: 2010-10-27t11
---

# boots &amp; background jobs - 

## Description

this sample was put together with jaykul’s help and bits and pieces were taken from the sample.ps1 distributed with powerboots. it requires the current changeset of powerboots. not the 0.2 beta.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 Import-Module PowerBoots
 
 Function global:Start-FakeDownload {
 	$global:job = Start-Job {
       $max = 50
 		foreach ($i in $(1..$max)){
 		}
 	}
 }
 
 $global:Window = Boots -Width 250 -Async -Passthru -Title "Progress Meter" {
 	DockPanel  {
 		ProgressBar -Height 25 -Name "Progress" -Dock Top | tee -var global:progress
 		Statusbar { Textblock | Tee -var global:status } -Dock Bottom 
 		WrapPanel { Button "Download" -Name "Download" | tee -var global:download } -horizontalalignment center
 	} -LastChildFill
 }
 
 $download.Add_Click({
 	$download.IsEnabled = $false
 	$download.Content = "Downloading..."
 	$updateblock = {
 		if($($job.State -eq "Running") -or $($($job.State -eq "Completed") -and $($(Receive-Job $job -Keep)[-1] -eq 100))){
 			Invoke-BootsWindow $Window {
 				$progress.Value = $(Receive-Job $job -Keep)[-1]
 				$status.Text = "$($(Receive-Job $job)[-1])`% done"
 			}
 		}
 		if($($job.State -eq "Completed") -and $($(Receive-Job $job) -eq $null)){
 			Invoke-BootsWindow $Window {
 				$status.Text = "Download Complete"
 			}
 			$timer.Stop()
 			$download.Content = "Download"
 			$download.IsEnabled = $true
 		}
 	}
 	$timer = new-object System.Windows.Threading.DispatcherTimer
 	$timer.Interval = [TimeSpan]"0:0:3"
 	$timer.Add_Tick( $updateBlock )
 	Start-FakeDownload 
 	$timer.start()
 })
`

