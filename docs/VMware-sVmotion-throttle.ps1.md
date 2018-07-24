---
Author: monahancj
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4880
Published Date: 2014-02-04t16
Archived Date: 2014-02-07t23
---

# vmware svmotion throttle - 

## Description

wait function to pause a script loop until the number of svmotions is below the specified number.  for vmware, but easy to convert i would think.

## Comments



## Usage



## TODO



## script

`wait-mtaskvmotions`

## Code

`#
 #
 <#
 	.SYNOPSIS
 		Will enter a wait/loop if there are running vMotion or svMotion tasks.
 	.DESCRIPTION
 		Checks the tasks and counts the number running vMotion or svMotion tasks.
 		If the number of active tasks is greater than the specified limit
 		then the function sleeps and checks again. The intended is as a throttle
 		in large (s)vMotion loops to prevent overloading the environment.  Is
 		especially useful in throttling a script while an unrelated event happens, 
 		such as putting a host or datastore into maintenance mode.
 	.PARAMETER vMotionLimit
 		The active tasks to test for.  If the number of active tasks is less than
 		this the function will exit.  The default is 1, and must be an integer.
 	.PARAMETER DelayMinutes
 		How long to wait before checking the active tasks again.  The default 
 		is 1, and must be an integer.
 	.EXAMPLE
 		Wait-mTaskvMotions -Verbose
 	.EXAMPLE
 		Wait-mTaskvMotions -vMotionLimit 4
 	.EXAMPLE
 		Wait-mTaskvMotions -vMotionLimit 2 -DelayMinutes 3 -Verbose
 	.NOTES
 		Using -Verbose will output standard script started and finished messages, 
 		and how long the function will sleep before it checks the active tasks again.
 	.LINK
 		http://mongit201.be.monster.com/chrism/virtech/blob/master/Repo/Wait-mTaskvMotions.ps1
 #>
 
 function Wait-mTaskvMotions {
 [CmdletBinding()]
 Param(
 	[int] $vMotionLimit=1,
 	[int] $DelayMinutes=5
 )
 
 $NumvMotionTasks = (Get-Task | ? { ($_.PercentComplete -ne 100) -and ( ($_.Description -like '*DRS*') -or ($_.Description -like '*vMotion*') )} | Measure-Object).Count
 
 While ( $NumvMotionTasks -ge $vMotionLimit ) {
   Write-Verbose "$(Get-Date)- Waiting $($DelayMinutes) minute(s) before checking again."
   Start-Sleep ($DelayMinutes * 60)
   $NumvMotionTasks = (Get-Task | ? { ($_.PercentComplete -ne 100) -and ( ($_.Description -like '*DRS*') -or ($_.Description -like '*vMotion*') )} | Measure-Object).Count
 }
 Write-Verbose "$(Get-Date)- Proceeding."
`

