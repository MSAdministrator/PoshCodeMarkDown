---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1471
Published Date: 
Archived Date: 2010-01-17t22
---

# get-crystalreporttable - 

## Description

find the tables used in a crystal report.

## Comments



## Usage



## TODO



## function

`get-crystalreporttable`

## Code

`#
 #
 function Get-CrystalReportTable()
 {
     <#
     .Synopsis
         Examines a Crystal Report and returns the tables used
     .Description
         Examines a Crystal Report and returns the tables used by the main report and subreports
     .Example
         $reports = Dir *.rpt | Get-CrystalReportTable
     .Notes
         Written by Steven Murawski on 11/12/09
         http://blog.usepowershell.com
         Version 0.1
     #>
     [cmdletbinding()]
     param (
         [parameter(ValueFromPipelineByPropertyName=$true,Mandatory=$true)]
         [Alias('FullName')]
         [string]$Path
     )
     begin
     {
         [reflection.assembly]::LoadWithPartialName('CrystalDecisions.Shared') | Out-Null
         [reflection.assembly]::LoadWithPartialName('CrystalDecisions.CrystalReports.Engine') | Out-Null
     }
     process
     {
         try
         {
             $output = New-Object PSObject | Select-Object Name, Tables, SubReports
             $output.name = Split-Path $path -Leaf
             
             $report = New-Object CrystalDecisions.CrystalReports.Engine.ReportDocument 
             $report.load($path)
             $output.Tables = $report.Database.Tables | Select-Object -expand Name
             
             
             $subreports = @($report.ReportDefinition.ReportObjects | Where-Object {$_ -is [CrystalDecisions.CrystalReports.Engine.SubreportObject]})
             if ($subreports.count -gt 0 )
             {
                 $output.subreports = @()
                 foreach ($subreport in $subreports)
                 {
                     $subreportdata = New-Object PSObject | Select-Object Name, Tables
                     $subreportdata.name = $subreport.SubreportName
                     
                     $subreportobject = $report.OpenSubreport($subreport.SubreportName)
                     $subreportdata.Tables = $subreportobject.Database.Tables | Select-Object -expand Name
                     $output.subreports += $subreportdata
                 }
             }          
             Write-Output $output
         }
         finally
         {
             $report.dispose()   
         }
     }
 }
`

