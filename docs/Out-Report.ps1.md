---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 8.5
Encoding: ascii
License: cc0
PoshCode ID: 1015
Published Date: 2010-04-11t11
Archived Date: 2016-03-05t02
---

# out-report - 

## Description

sends output to excel, pdf, or image files

## Comments



## Usage



## TODO



## function

`new-imagedeviceinfo`

## Code

`#
 #
 
 param($fileName,$fileType,$imageType,$height=11,$width=8.5)
 
 $libraryDir = Convert-Path (Resolve-Path "$ProfileDir\Libraries")
 [void][reflection.assembly]::LoadWithPartialName("Microsoft.ReportViewer.WinForms")
 [void][Reflection.Assembly]::LoadFrom("$libraryDir\ReportExporters.Common.dll")
 [void][Reflection.Assembly]::LoadFrom("$libraryDir\ReportExporters.WinForms.dll")
 
 $fileTypes = 'XLS','PDF','IMAGE'
 if (!($fileTypes -contains $fileType)) 
 { throw 'Valid file types are XLS, PDF, IMAGE' }
 
 $imageTypes = 'BMP','EMF','GIF','JPEG','PNG','TIFF'
 if ( $imageType -and !($imageTypes -contains $imageType)) 
 { throw 'Valid image types are BMP,EMF,GIF,JPEG,PNG or TIFF' }
 
 #######################
 function New-ImageDeviceInfo
 {
     param($imageType,$height,$width)
 
     $deviceInfo = new-object ("ReportExporters.Common.Exporting.ImageDeviceInfoSettings") $imageType
     $deviceInfo.PageHeight = new-object System.Web.UI.WebControls.Unit($height,[System.Web.UI.WebControls.UnitType]::Inch)
     $deviceInfo.PageWidth = new-object System.Web.UI.WebControls.Unit($width,[System.Web.UI.WebControls.UnitType]::Inch)
     $deviceInfo.StartPage = 0
     return $deviceInfo
 
 
 $dt = new-Object Data.datatable  
   $First = $true  
   foreach ($item in $input){  
     $DR = $DT.NewRow()  
     $Item.PsObject.get_properties() | foreach {  
       If ($first) {  
         $Col =  new-object Data.DataColumn  
         $Col.ColumnName = $_.Name.ToString()  
         $DT.Columns.Add($Col)       }  
       if ($_.value -eq $null) {  
         $DR.Item($_.Name) = "[empty]"  
       }  
       ElseIf ($_.IsArray) {  
         $DR.Item($_.Name) =[string]::Join($_.value ,";")  
       }  
       Else {  
         $DR.Item($_.Name) = $_.value  
       }  
     }  
     $DT.Rows.Add($DR)  
     $First = $false  
 
 $ds = new-object System.Data.dataSet 
 $ds.merge($dt)
 $dsaProvider = new-object ReportExporters.Common.Adapters.DataSetAdapterProvider $ds
 $dsa = $dsaProvider.GetAdapters()
 $reportExporter = new-object ReportExporters.WinForms.WinFormsReportExporter $dsa
 
 switch ($fileType)
 {
     'XLS'   { $content =$reportExporter.ExportToXls() }
     'PDF'   { $content = $reportExporter.ExportToPdf() }
     'IMAGE' { $deviceInfo = New-ImageDeviceInfo $imageType $height $width; $content =  $reportExporter.ExportToImage($deviceInfo) }
 }
 
 [System.IO.File]::WriteAllBytes($fileName,$content.ToArray())
`

