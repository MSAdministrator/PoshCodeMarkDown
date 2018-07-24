---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1977
Published Date: 
Archived Date: 2016-02-04t04
---

#  - 

## Description

use ssrs reportviewer with powershell, use parameters and catch navigate event

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
             
             
             
  [System.Reflection.Assembly]::Load("Microsoft.ReportViewer.WinForms, Version=10.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a")            
             
              
 $rv = New-Object Microsoft.Reporting.WinForms.ReportViewer            
             
 $eventHandler = [Microsoft.Reporting.WinForms.HyperlinkEventHandler]{            
     Write-Host "Hyperlink Event fired"            
     $_.Cancel = $true                   
     }             
 $rv.Add_Hyperlink($eventHandler)             
             
 $rv.ProcessingMode = "Remote"            
 $rv.ServerReport.ReportServerUrl = "http://localhost/reportserver"            
 $rv.ServerReport.ReportPath = "/Reports/Sample Report"            
              
 #$rv.ServerReport.ReportServerCredentials.NetworkCredentials=             
              
 $rv.Height = 800            
 $rv.Width = 1200            
 $rv.Dock = [System.Windows.Forms.DockStyle]::Fill            
             
             
 $params = new-object 'Microsoft.Reporting.WinForms.ReportParameter[]' 1            
 $params[0] = new-Object Microsoft.Reporting.WinForms.ReportParameter("Par1", "3159", $false)            
             
 $rv.ServerReport.SetParameters($params);            
             
             
             
 $rv.ShowParameterPrompts = $false            
 $rv.RefreshReport()            
              
 #--------------------------------------------------------------            
 #--------------------------------------------------------------            
 $form = New-Object Windows.Forms.Form            
              
 $form.Height = 810            
 $form.Width= 1210            
 $form.Controls.Add($rv)            
 $rv.Show()            
 $form.ShowDialog()            
             
             
 <#
 http://blogs.msdn.com/b/powershell/archive/2008/05/24/wpf-powershell-part-3-handling-events.aspx
 http://social.msdn.microsoft.com/Forums/en-US/vsreportcontrols/thread/693694bc-bfbc-4786-98ee-a99826e78739
 #>
`

