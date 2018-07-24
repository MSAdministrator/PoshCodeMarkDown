---
Author: rob sewell http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5033
Published Date: 2014-03-31t20
Archived Date: 2014-04-03t03
---

# sql agent jobs to excel - 

## Description

name

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
  #############################################################################################
 #
 #
 #
 #
 
 $xl = new-object -comobject excel.application
 
 $wb = $xl.Workbooks.Add()
 $ws = $wb.Worksheets.Item(1)
 
 $date = Get-Date -format f
 $Filename = ( get-date ).ToString('ddMMMyyyHHmm')
 
 $cells=$ws.Cells
 
 
 $cells.item(1,3).font.bold=$True
 $cells.item(1,3).font.size=18
 $cells.item(1,3)="Back Up Report $date"
 $cells.item(5,9)="Last Job Run Older than 1 Day"
 $cells.item(5,8).Interior.ColorIndex = 43
 $cells.item(4,9)="Last Job Run Older than 7 Days"
 $cells.item(4,8).Interior.ColorIndex = 53
 $cells.item(7,9)="Successful Job"
 $cells.item(7,8).Interior.ColorIndex = 4
 $cells.item(8,9)="Failed Job"
 $cells.item(8,8).Interior.ColorIndex = 3
 $cells.item(9,9)="Job Status Unknown"
 $cells.item(9,8).Interior.ColorIndex = 15
 
 
 $row=3
 $col=2
 
 
     $cells.item($row,$col)="Server"
     $cells.item($row,$col).font.size=16
     $Cells.item($row,$col).Columnwidth = 10
     $col++
     $cells.item($row,$col)="Job Name"
     $cells.item($row,$col).font.size=16
     $Cells.item($row,$col).Columnwidth = 40
     $col++
     $cells.item($row,$col)="Enabled?"
     $cells.item($row,$col).font.size=16    
     $Cells.item($row,$col).Columnwidth = 15
     $col++    
     $cells.item($row,$col)="Outcome"
     $cells.item($row,$col).font.size=16
     $Cells.item($row,$col).Columnwidth = 12
     $col++
     $cells.item($row,$col)="Last Run Time"
     $cells.item($row,$col).font.size=16    
     $Cells.item($row,$col).Columnwidth = 15
     $col++
 
    
 [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.Smo") | Out-Null;
 
 $sqlservers = Get-Content 'D:\SkyDrive\Documents\Scripts\Powershell Scripts\sqlservers.txt';
 
 foreach($sqlserver in $sqlservers)
 {
       $srv = New-Object "Microsoft.SqlServer.Management.Smo.Server" $sqlserver;
  
       foreach($job in $srv.JobServer.Jobs)
       {
 
             $jobName = $job.Name;
             $jobEnabled = $job.IsEnabled;
             $jobLastRunOutcome = $job.LastRunOutcome;
             $Time = $job.LastRunDate ;
 
             if($jobEnabled -eq "FALSE")
             { $colourenabled = "2"}
             else {$colourenabled = "48" }         
 
             if($jobLastRunOutcome -eq "Failed")
             }
             
             Elseif($jobLastRunOutcome -eq "Unknown")
 
     $row++
     $col=2
     $cells.item($Row,$col)=$sqlserver
     $col++
     $cells.item($Row,$col)=$jobName
     $col++
     $cells.item($Row,$col)=$jobEnabled    
     
     $cells.item($Row,$col).Interior.ColorIndex = $colourEnabled
         if ($colourenabled -eq "48")        
     { 
         $cells.item($Row ,1 ).Interior.ColorIndex = 48
         $cells.item($Row ,2 ).Interior.ColorIndex = 48
         $cells.item($Row ,3 ).Interior.ColorIndex = 48
         $cells.item($Row ,4 ).Interior.ColorIndex = 48
         $cells.item($Row ,5 ).Interior.ColorIndex = 48
         $cells.item($Row ,6 ).Interior.ColorIndex = 48
         $cells.item($Row ,7 ).Interior.ColorIndex = 48
         } 
     $col++
 
     $cells.item($Row,$col)="$jobLastRunOutcome"
     $cells.item($Row,$col).Interior.ColorIndex = $colour
 
     if ($colourenabled -eq "48") 
     {$cells.item($Row,$col).Interior.ColorIndex = 48}
 
     $col++
 
     $cells.item($Row,$col)=$Time 
     
 
     If($Time -lt ($(Get-Date).AddDays(-1)))
     { $cells.item($Row,$col).Interior.ColorIndex = 43}
         If($Time -lt ($(Get-Date).AddDays(-7)))
             { $cells.item($Row,$col).Interior.ColorIndex = 53} 
               
     }
     $row++
     $row++
 
     $ws.rows.item($Row).Interior.ColorIndex = 6
     $row++
     $ws.rows.item($Row).Interior.ColorIndex = 6
     $row++
     }
 
 
 $wb.Saveas("C:\temp\Test$filename.xlsx")
 $xl.quit()
 Stop-Process -Name EXCEL
`

