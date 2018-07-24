---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6219
Published Date: 
Archived Date: 2016-03-19t00
---

#  - 

## Description

function that return the console output of all running amazon instances

## Comments



## Usage



## TODO



## function

`get-ec2runninginstancesconsoleoutput`

## Code

`#
 #
 Ipmo 'C:\Program Files\AWS Tools\PowerShell\AWSPowerShell\AWSPowerShell.psd1'
 
 function Get-EC2RunningInstancesConsoleOutput ($AccessKey, $SecretKey)
 {
     $RunningFilter = new-object Amazon.EC2.Model.Filter
     $RunningFilter.Name = 'instance-state-name'
     $RunningFilter.Value = 'running'
     $instances = Get-EC2Instance -accesskey $accessKey -secretkey $secretkey -Filter $RunningFilter
     $RunningInstanceID = $instances.Instances.instanceID
     foreach ($instanceID in $RunningInstanceID)
         {
         $ConsoleOut = Get-EC2ConsoleOutput -AccessKey $accessKey -secretkey $secretKey -InstanceID $instanceID
         $output = [System.Convert]::FromBase64String($ConsoleOut.Output.ToString())
         $output = [System.Text.Encoding]::UTF8.GetString($output)
         $line = "" | select instanceID, consoleLine
         $output.split([char]10) | % {$line.instanceID = $instanceID;$line.consoleLine=$_;$line}
         }
 }
`

