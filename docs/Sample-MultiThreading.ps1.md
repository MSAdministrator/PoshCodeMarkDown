---
Author: autom8
Publisher: 
Copyright: 
Email: 
Version: 172.16.1
Encoding: ascii
License: cc0
PoshCode ID: 6739
Published Date: 2017-02-19t02
Archived Date: 2017-05-11t14
---

# sample multithreading - 

## Description

an example of multi threading in powershell

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 $CSVFileName = "\\networkshare\folder\filename.csv"
 "CSV,Header,With,Well,Named,Data,Columns" > $CSVFileName
 
 $Timestamp = get-date -format o 
 
 $LogFileName = "\\networkshare\folder\logfile_$Timestamp.log"
 
 $SomeFunction = {
 	param($IPAddress,$CSVFileName,$LogFileName)
 	
 
     Sleep -s 1
     
     $CSVOutput = "Hello,$IPAddress,My,Name,Is,Mr,Bean"
     $CSVOutput >> $CSVFileName
 
     $error >> $LogFileName
 }
 
 $Subnets = @("192.168.1.","192.168.2.","172.16.1.")
 
 $Networks=@()
 
 foreach($Subnet in $Subnets)
 {
     $Network=@()
 
     $IPs = 1..254
     
     foreach($IP in $IPs){$Network+=($Subnet + $IP)}
 
     $Networks += $Network
 }
 
 $NumIPaddresses = 0
 foreach($Network in $Networks)
 {
     foreach($IP in $Network){$NumIPaddresses++}
 }
 
 
 $NumThreads = 20
 $ThreadsProcessed = 0
 
 foreach($Network in $Networks)
 {
     foreach($IP in $Network)
     {
         $NumJobsRunning = get-job -State Running
         while($NumJobsRunning.Count -ge $NumThreads)
         {
             Sleep -s 30
             $NumJobsRunning = get-job -State Running
         }
 
 	    Start-Job -ScriptBlock $SomeFunction -ArgumentList $IP,$CSVFileName,$LogFileName
         $ThreadsProcessed++
         
         $Status = "$ThreadsProcessed of $NumIPAddresses threads processed"
         echo $Status
     }
 }
 
 $CSVData = Import-CSV $CSVFileName
 $CSVData | Out-gridview
`

