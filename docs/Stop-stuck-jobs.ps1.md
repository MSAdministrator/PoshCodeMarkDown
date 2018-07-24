---
Author: autom8
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6741
Published Date: 2017-02-19t05
Archived Date: 2017-02-23t01
---

# stop stuck jobs - 

## Description

stops stuck jobs and gets info from them

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $ThreadTimeout = 300
 
 $RunningJobs = get-job -State Running
 foreach($RunningJob in $RunningJobs)
 {
     $CurrentTime = get-date
     $TimeoutTime = $RunningJob.PSBeginTime
     $TimeoutTime = $TimeoutTime.AddSeconds($ThreadTimeout)
 
     if($CurrentTime -gt $TimeoutTime)
     {
         $Log = @()
         $Log += ("Killing stuck thread " + $RunningJob.Name)
         $Log += $RunningJob.Output
         $Log += $RunningJob.Error
 
         echo $Log
         $Log >> $LogFileName
 
         $RunningJob | Stop-Job
     }                
 }
`

