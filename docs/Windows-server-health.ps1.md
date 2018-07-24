---
Author: prashant pandey
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6558
Published Date: 2017-10-07t07
Archived Date: 2017-04-30t12
---

# windows server health - 

## Description

windows server health check for 2008 2012 2012r2

## Comments



## Usage



## TODO



## class

`get-memoryusage`

## Code

`#
 #
 ########################################################
 
 ########################################################
 ########################################################
 
 
 
 
 $Outputreport=""
 $Outputreport +="<style>TABLE{ border-width: 1px;border-style: solid;border-color: black;border-collapse: collapse;align:center;width:100%;}
 TH{border-width: 1px;background-color: lightblue;bgcolor=blue;padding: 3px;border-style: solid;border-color: black;}
 TD{border-width: 1px;color: white;background-color: gray;padding: 3px;border-style: solid;border-color: black;}
  
 
 
 
 $Hostname = hostname | Out-String
 
 $Version = (Get-WmiObject -class Win32_OperatingSystem).Caption | Out-String
 
 $UPTIME=Get-WmiObject Win32_OperatingSystem
 $up = [Management.ManagementDateTimeConverter]::ToDateTime($UPTIME.LastBootUpTime) | Out-String
 
 $Disk = Get-WmiObject Win32_logicaldisk -ComputerName LocalHost -Filter "DriveType=3" |select -property DeviceID,@{Name="Size(GB)";Expression={[decimal]("{0:N0}" -f($_.size/1gb))}},@{Name="Free Space(GB)";Expression={[decimal]("{0:N0}" -f($_.freespace/1gb))}},@{Name="Free (%)";Expression={"{0,6:P0}" -f(($_.freespace/1gb) / ($_.size/1gb))}}|ConvertTo-Html
 
 $Private:wmiService =gsv -include "*SQL*" -Exclude "*ySQL*","*spo*"|select Name,DisplayName,Status|ConvertTo-Html
 $Services =gsv -include "*SQL*" -Exclude "*ySQL*","*spo*"|select Name,DisplayName,Status|ConvertTo-Html 
 
 $CPU_Utilization = Get-Process|Sort-object -Property CPU -Descending| Select -first 5 -Property ID,ProcessName,@{Name = 'CPU In (%)';Expression = {$TotalSec = (New-TimeSpan -Start $_.StartTime).TotalSeconds;[Math]::Round( ($_.CPU * 100 /$TotalSec),2)}},@{Expression={$_.threads.count};Label="Threads";},@{Name="Mem Usage(MB)";Expression={[math]::round($_.ws / 1mb)}},@{Name="VM(MB)";Expression={"{0:N3}" -f($_.VM/1mb)}}|ConvertTo-Html
 #$proc =get-counter -Counter "\Processor(_Total)\% Processor Time" -SampleInterval 2 
 #$cpu=($proc.readings -split ":")[-1]
 #$CPU_Utilization = [System.Math]::Round($cpu, 2) | Out-String
  
 
 $arr=@()
 $ProcessorObject=gwmi win32_processor
 foreach($processor in $ProcessorObject)
 {
    $arr += $processor.Caption
    $arr += $processor.LoadPercentage
 }
 
 $SecPatch = get-hotfix -Description "Security Update" |sort "Description" -desc | select Description,installedon -first 1 | Out-String
 
 $Private:perfmem = Get-WmiObject -namespace root\cimv2 Win32_PerfFormattedData_PerfOS_Memory
 $Private:totmem = Get-WmiObject -namespace root\cimv2 CIM_PhysicalMemory 
 [Int32]$Private:totalcapacity = 0 
 foreach ($Mem in $totmem) 
 { 
 $totalcapacity += $Mem.Capacity / 1Mb 
 } 
 
 $Private:tmp = New-Object -TypeName System.Object 
 $tmp | Add-Member -Name CapacityMB -Value $totalcapacity -MemberType NoteProperty 
 $tmp | Add-Member -Name AvailableMB -Value $perfmem.AvailableMBytes -MemberType NoteProperty
 $ram_usage = $tmp |ConvertTo-Html
 
 function Get-MemoryUsage ($ComputerName=$ENV:ComputerName) {
 if (Test-Connection -ComputerName $ComputerName -Count 1 -Quiet) {
 $ComputerSystem = Get-WmiObject -ComputerName $ComputerName -Class Win32_operatingsystem -Property TotalVisibleMemorySize, FreePhysicalMemory
 $MachineName = $ComputerSystem.CSName
 $FreePhysicalMemory = ($ComputerSystem.FreePhysicalMemory) / (1mb)
 $TotalVisibleMemorySize = ($ComputerSystem.TotalVisibleMemorySize) / (1mb)
 $TotalVisibleMemorySizeR = "{0:N2}" -f $TotalVisibleMemorySize
 $TotalFreeMemPerc = ($FreePhysicalMemory/$TotalVisibleMemorySize)*100
 $TotalFreeMemPercR = "{0:N2}" -f $TotalFreeMemPerc
 "<table border=1 width=100>"
 "<tr><th>RAM</th><td>$TotalVisibleMemorySizeR GB</td></tr>"
 "<tr><th>Free Physical Memory</th><td>$TotalFreeMemPercR %</td></tr>"
 "</table>"
 
 }}
 $PhyMem = Get-MemoryUsage
 $Hotfix=(get-hotfix | sort installedon)|select -first 5 HotFixID,InstalledBy,InstalledOn,Description|ConvertTo-Html
 $Processor_Counter=Get-Counter "\Processor(_total)\% Processor Time"|ConvertTo-Html
 $Total_Threads=(Get-Process |Select-Object -ExpandProperty Threads).Count
 
 function Get-PageFile { 
 param( 
     [string]$computer="." 
 )    
         Get-WmiObject -Class Win32_PageFileUsage  -ComputerName $computer | 
         Select  @{Name="File";Expression={ $_.Name }}, 
         @{Name="Base Size(MB)"; Expression={$_.AllocatedBaseSize}}, 
         @{Name="Peak Size(MB)"; Expression={$_.PeakUsage}},  
         TempPageFile 
   }
   
 $PhysicalRAM = (Get-WMIObject -class Win32_PhysicalMemory  |
 Measure-Object -Property capacity -Sum | % {[Math]::Round(($_.sum / 1GB),2)})
 $ht = @{}
 $ht.Add('Total_Ram(GB)',$PhysicalRAM)
 $OSRAM = gwmi Win32_OperatingSystem  |
 foreach {$_.TotalVisibleMemorySize,$_.FreePhysicalMemory}
 $ht.Add('Total Visable RAM GB',([Math]::Round(($OSRAM[0] /1MB),4)))
 $ht.Add('Available_Ram(GB)',([Math]::Round(($OSRAM[1] /1MB),4)))
 $RAM = New-Object -TypeName PSObject -Property $ht|ConvertTo-Html
 $Paging1=Get-PageFile|ConvertTo-Html
 $Paging =  Get-WMIObject Win32_PageFileSetting |  select Name, InitialSize, MaximumSize|ConvertTo-Html
 
 $Available_Bytes=Get-Counter -Counter "\Memory\Available Bytes"|Select -ExpandProperty CounterSamples|Select CookedValue |ft -HideTableHeaders|out-string
 $att=Get-Counter -Counter "\Memory\Committed Bytes"|Select -ExpandProperty CounterSamples|Select CookedValue |ft -HideTableHeaders|out-string
 $Comitted_Bytes="{0:N0}" -f (($att.trim())/1024/1024)
 $Handle_Count=Get-Counter -Counter "\Process(_Total)\Handle Count"|Select -ExpandProperty CounterSamples|Select CookedValue |ft -HideTableHeaders|out-string
 $Thread_Count=Get-Counter -Counter "\Process(_Total)\Thread Count"|Select -ExpandProperty CounterSamples|Select CookedValue |ft -HideTableHeaders|out-string
 $ftt=Get-Counter -Counter "\memory\Pool Paged Bytes"|Select -ExpandProperty CounterSamples|Select CookedValue |ft -HideTableHeaders|out-string
 $Pool_Paged="{0:N0}" -f (($ftt.trim())/1024/1024)
 $dtt=Get-Counter -Counter "\memory\Pool Nonpaged Bytes"|Select -ExpandProperty CounterSamples|Select CookedValue |ft -HideTableHeaders|out-string
 $Pool_NonPaged="{0:N0}" -f (($dtt.trim())/1024/1024)
 $Total_process=(get-process).count
 
 $Outputreport +="<BODY><HTML>"
 
 $Outputreport +="<table border=1 ><tr><td>"
 $Outputreport +="<table border=1 width=100%>"
 $Outputreport +="<tr><th><B>Hostname</B></th><td>"+$Hostname+"</td></tr>"
 $Outputreport +="<tr><th><B>Version</B></th><td>"+$Version+"</td></tr>"
 $Outputreport +="<tr><th><B>Uptime</B></th><td>"+$up+"</td></tr>"
 $Outputreport +="<tr><th><B>Physical Memory(MB)</B></th><td>"+$RAM+"</td></tr></td></tr><tr><td><tr><th><B>System</B></th></tr><tr><th>Total Handles</th><td>"+$Handle_Count.trim()+"</td></tr><tr><th>Total Thread</th><td>"+$Thread_Count.trim()+"</td></tr><tr><th>Total Process</th><td>"+$Total_process+"</td></tr><tr><th>Commit(MB)</th><td>"+$Comitted_Bytes.trim()+"</td></tr></td>"
 $Outputreport +="<td><tr><th><B>Kernel Memory(MB)</B></th></tr><tr><th>Paged</th><td>"+$Pool_Paged.trim()+"</td></tr><tr><th>Non Paged</th><td>"+$Pool_NonPaged.trim()+"</td></tr></td></tr></table>"
 $Outputreport += "</table></BODY></HTML>"
 $Outputreport +="</br>"
 $Outputreport +="</br>"
 $Outputreport +="<BODY><HTML>"
 $Outputreport +="<table border=1 width=50%>"
 $Outputreport +="<tr><th><B>Disk Size</B></th><td>"+$Disk+"</td></tr>"
 $Outputreport +="<tr><th><B>Services</B></th><td>"+$Services+"</td></tr>"
 $Outputreport +="<tr><th><B>Top5 Process</B></th><td>"+$CPU_Utilization+"</td></tr>"
 $Outputreport +="<tr><th><B>Ram_Usage</B></th><td>"+$ram_usage+"</td></tr>"
 $Outputreport +="<tr><th><B>Physical Memory</B></th><td>"+$PhyMem+"</td></tr>"
 #$Outputreport +="<tr><th><B>Processor Counter</B></th><td>"+$Processor_Counter+"</td></tr>"
 $Outputreport +="<tr><th><B>Last 5 HotFix</B></th><td>"+$Hotfix+"</td></tr>"
 $Outputreport +="<tr><th><B>Paging</B></th><td>"+$Paging+"</td></tr>"
 #$Outputreport +="<tr><th><B>No Of Threads</B></th><td>"+$Total_Threads+"</td></tr>"
 #$Outputreport +="<tr><th><B>Average process</B></th><table border=1><tr><th>Cpu Load</th><th>Mem Load</th><th>$CDrive</th></tr><tr><td>"+$CPULoad+"</td><td>"+$MemLoad+"</td><td>"+$CDrive+"</td></tr></table></tr>"
 $Outputreport += "</table></BODY></HTML>"
 $Outputreport | out-file C:\Scripts\Windows_Server_Health_Status.html
`

