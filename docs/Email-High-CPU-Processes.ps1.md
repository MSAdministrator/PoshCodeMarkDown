---
Author: anonymous
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4505
Published Date: 2014-10-04t15
Archived Date: 2017-01-29t16
---

# email high cpu processes - 

## Description

win32_process doesn’t have %cpu usage so this queries win32_perfformatteddata_perfproc_process for high cpu processes. then queries the processes to get owner information and other properties.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $CustObj = @()
 
 $HighProcPids = gwmi Win32_PerfFormattedData_PerfProc_Process | ?{(($_.percentprocessortime -ne 0) -and ($_.idprocess -ne 0))}| Select IdProcess, PercentProcessorTime
 
 foreach($obj in $HighProcPids){
     $tmpProcess = gwmi Win32_Process -Filter ([string]::Format("ProcessId=`'{0}`'",$obj.IdProcess))
     $tmpObj = New-Object System.Object
     Add-Member -InputObject $tmpObj -MemberType NoteProperty -Name "Owner" -Value $tmpProcess.getowner().user
     Add-Member -InputObject $tmpObj -MemberType NoteProperty -Name "PID" -Value $obj.IdProcess
     Add-Member -InputObject $tmpObj -MemberType NoteProperty -Name "Name" -Value $tmpProcess.ProcessName
     Add-Member -InputObject $tmpObj -MemberType NoteProperty -Name "CPU" -Value $obj.PercentProcessorTime
     Add-Member -InputObject $tmpObj -MemberType NoteProperty -Name "Cmdline" -Value $tmpProcess.CommandLine
     $CustObj += $tmpObj
 }
 
 Send-MailMessage -SmtpServer "smtp.domain.com" -To "admin@domain.com" -From ($env:computername+"@domain.com") -Subject "High CPU on $env:computername" -Body ($CustObj | Format-List * | Out-String)
 Exit
`

