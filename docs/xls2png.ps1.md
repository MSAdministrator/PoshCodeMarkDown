---
Author: egil aspevik
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 5184
Published Date: 2014-05-23t14
Archived Date: 2014-05-26t16
---

# xls2png - 

## Description

bulk converts xls ranges to png by using the clipboard. reads config from xml. now with working code…

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
 Bulk export Excel worksheet ranges to images (PNG)
 
 .DESCRIPTION
 This script exports a range (ex. A5:L20) in a worksheet (ex. Sheet1) in a workbook (ex. c:\temp\myfile.xlsx) to a PNG file 
 
 by using the clipboard.
 Do NOT use the clipboard during the execution of this script.
 It relies on the Excel COM objects which again requires that Excel is installed.
 The script read its config from an XML file formatted like this:
 
 <XLSPIC>  
      <xlfile value="c:\xlsexport\file1.xls">
          <sheet value="Sheet1">
              <range value="B7:L55" />
              <range value="P1:Q20" />
          </sheet>
          <sheet value="Name-of-my-sheet2">
              <range value="A3:S24" />
          </sheet>
      </xlfile>
      <xlfile value="c:\xlsexport\file2.xlsx">
          <sheet value="A very large sheet">
              <range value="B7:B6300" />
          </sheet>
      </xlfile>
 </XLSPIC>
 
 
 The PNG files are saved like this: workbookfilename-sheetname-range.png
 
 This script will probably need some tuning on other languages and/or powershell versions. Tested with PS 3.0 and Excel 
 
 2010 on Win 7 64bit EN-US.
 
 If Excel is in another culture than powershell, you will most likely run into problems. A typical error is the "Old format 
 
 or invalid type library" - see http://support.microsoft.com/kb/320369
 
 Author: Egil Aspevik (egilaspevikATgmail)
 
 .PARAMETER savetodir
 The directory to save the PNG files to
 
 .PARAMETER configfile
 The XML config file as explained in the description
 
 .PARAMETER forceexcelclose
 By default, the script do not force-close Excel before starting to parse. This parameter does just that to ensure that the 
 
 COM object gets a clean start.
 
 .LINK 
 http://support.microsoft.com/kb/320369
 #>
 
 
 
 
 Param([string]$savetodir="c:\xlsexport",[string]$configfile="c:\xlsexport\config.xml", [switch]$forceexcelclose=$false)
 
 $ErrorActionPreference="stop"
 
 
 if ($forceexcelclose) { Get-Process EXCEL -erroraction silentlycontinue | Stop-Process -Force }
 
 if (-not (test-path $configfile)) {
   write-error "Configfile not found. Use -configfile"
   exit 1
 
 }
 
 if (-not (Test-Path $savetodir)) {
   write-error "Path not found. Create one and use -savetodir"
   exit 1
 }
 try {
   $cfg=[xml] (cat $configfile)
   }
 catch {
   write-error "Couldn't parse $configfile. Exception message: $_"
   exit 1
 
 }
 
 $cfg.XLSPIC.xlfile.value | % { if (-not (test-path $_)) { write-error "Couldn't access file: $_. Check your XML"; exit 1 } 
 
 }
 
 
   
 try {
 add-type -an system.windows.forms 
 } catch {
   write-error "Couldn't add system.windows.forms. You shouldn't see this. Ever. Exception msg: $_"
   exit 1
 }
 
 
 try {
 $xl=New-Object -ComObject Excel.Application
 $xl.visible=$false
 $xl.displayalerts=$false
 } catch {
   write-error "Couldn't open Excel. Excel MUST be installed on the executing computer. Exception message: $_"
   exit 1
 }
 
 try {
 
 $cfg.XLSPIC.xlfile | % {
 
     $excelfile=$_.value
    
     $wb=$xl.workbooks.open($excelfile)
     
     $_.sheet | % {
         $sheet=$_.value
         $sh=$wb.Worksheets.Item($sheet)
         $_.range | % {
             
             $range=$_.value
             $rng=$sh.range($range)
 
 (v=office.15).aspx
             $pic=$rng.CopyPicture(1,2)
 
 
             if (-not ([System.Windows.Forms.Clipboard]::ContainsImage())) {
               write-error "Couldn't copy $excelfile - $sheet - $range as a picture." -ErrorAction Continue
               
             }
             else {
                 $e=[System.Windows.Forms.Clipboard]::GetDataObject()
                 $img=$e.GetImage()
                 $basename=$excelfile -replace '.+\\(.+)','$1'
                 $rangename=$range -replace ':','TO'
                 $sheetname=$sheet -replace ':','_'
                 $savename=$savetodir+"\"+"$basename-$sheetname-$rangename"+".png"
                 
                 rm $savename -Force -ErrorAction SilentlyContinue
                 $img.save($savename)
                 "Processed $excelfile - $sheet - $range. Saved as $savename"
             }
         }
         
     }
     $wb.close()
 }
 
 $xl.Quit()
 if (([System.Runtime.Interopservices.Marshal]::ReleaseComObject($xl))) {  Write-Error "Couldn't release COM object. Please 
 
 kill excel process manually" }
 } catch {
 
  write-error "Stopped processing during or after: $excelfile, $sheet, $range. Check XML and so on. Exception message: $_"
  exit 1
 }
 
 exit 0
`

