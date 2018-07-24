---
Author: anonymous
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3999
Published Date: 2013-03-06t20
Archived Date: 2013-03-10t11
---

# export asp events 2 evtx - 

## Description

export all asp generated events in the application event log to a .evtx file. note that a separate file will be made for each “provider” or .net version installed.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $DstFolder = "D:\somefolder\"
 
 $EvtSession = New-Object System.Diagnostics.Eventing.Reader.EventLogSession($env:computername)
 
 [string[]] $ProviderList = $EvtSession.GetProviderNames() | Select-String asp
 
 for($i=0;$i -lt $ProviderList.Length;$i++){
     $EvtQuery = "*[System/Provider/@Name=`""+$ProviderList[$i]+"`"]"
     $Dst = $DstFolder+$env:computername+"_"+($ProviderList[$i]).replace(" ","_")+".evtx"
     if(Test-Path -Path $Dst){Remove-Item -Path $Dst -Force}
     $EvtSession.ExportLogAndMessages('Application','LogName',$EvtQuery,$Dst)
 }
`

