---
Author: steele stenger
Publisher: 
Copyright: 
Email: 
Version: 9.14
Encoding: ascii
License: cc0
PoshCode ID: 5084
Published Date: 2014-04-16t21
Archived Date: 2014-04-20t00
---

# office pass saveas - 

## Description

use known password cache to open and save unprotected copies of documents.   – work in progress

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $ErrorActionPreference = "SilentlyContinue"
 
 CLS
 
 $encrypted_path = "C:\Password_Recovery\Encrypted"
 $decrypted_Path = "c:\Password_Recovery\Decrypted\"
 $original_Path = "c:\Password_Recovery\Originals\"
 $password_Path = "c:\Password_Recovery\Passwords\Passwords.txt"
 
 $arrPasswords = Get-Content -Path $password_Path
 
 $arrFiles = Get-ChildItem $encrypted_path 
 
 [int] $count = ($arrfiles.count -1)
 
 $arrFiles| % {
     $file  = get-item -path $_.fullname 
     write-host "Processing" $file.name -f "DarkYellow"
     write-host "Items remaining: " $count `n
 
     if ($file.Extension -eq ".docx") {
 
         $arrPasswords | % {
             $passwd = $_
             
             $WordObj = $null
             $WordObj = New-Object -ComObject Word.Application
             $WordObj.Visible = $false
             $WordObj.Options.WarnBeforeSavingPrintingSendingMarkup = $false
 
             $WordDoc = $WordObj.Documents.Open($file.fullname, $null, $false, $null, $passwd, $passwd)
             $WordDoc.Activate()
             $content = $null
             $content = $WordDoc.content
 
             if ($content -ne $null) {
                 write-host "opened " -f "Yellow"
                 $WordDoc.Password=$null
                 $savePath = $decrypted_Path+$file.Name
                 write-host "Decrypted: " $file.Name -f "DarkGreen"
                 $WordDoc.SaveAs([ref]$savePath)
                 $WordDoc.Close()
                 $WordObj.Application.Quit()
 
                 move-item $file.fullname -Destination $original_Path -Force
                 }
             else {
                 $WordDoc.Close()
                 $WordObj.Application.Quit()
             }
         }
     }
     elseif ($file.Extension -eq ".xlsx") {
 
         $arrPasswords | % {
             $passwd = $_
             
             $ExcelObj = $null
             $ExcelObj = New-Object -ComObject Excel.Application
             $ExcelObj.Visible = $false
 
             $Workbook = $ExcelObj.Workbooks.Open($file.fullname,1,$false,5,$passwd)
             $Workbook.Activate()
 
                 if ($Workbook.Worksheets.count -ne 0) {
                     $Workbook.Password=$null
                     $savePath = $decrypted_Path+$file.Name
                     write-host "Decrypted: " $file.Name -f "DarkGreen"
                     $Workbook.SaveAs($savePath)
                     $ExcelObj.Workbooks.close()
                     $ExcelObj.Application.Quit()
 
                     move-item $file.fullname -Destination $original_Path -Force
                 }
                 else {
                     $ExcelObj.Close()
                     $ExcelObj.Application.Quit()
                 }
         }
 
     }
     
     elseif ($file.Extension -eq ".pptx") {
 
     
     $arrPasswords | % {
         $passwd = $_
 
         $objPPT = New-Object -ComObject Powerpoint.Application
 
         $Presentation = $objPPT.ProtectedViewWindows.Open($file, $passwd)
 
         if ($presentation.application -ne $null) {
             $openPresentationWithPasswords = $Presentation.Edit("ModifyPassword")
             $openPresentationWithPasswords.Password = $null
 
             $savePath = $decrypted_Path+$file.Name
             write-host $savepath -f "yellow"
             $openPresentationWithPasswords.SaveAs($savePath)
             write-host "Decrypted: " $file.Name -f "DarkGreen"
 
             $openPresentationWithPasswords.Close()
             $objPPT.Quit()
             [System.Runtime.Interopservices.Marshal]::ReleaseComObject($openPresentationWithPasswords) | out-null
             [System.Runtime.Interopservices.Marshal]::ReleaseComObject($presentation) | out-null
             [System.Runtime.Interopservices.Marshal]::ReleaseComObject($objPPT) | out-null
 
             move-item $file.fullname -Destination $original_Path -Force
         }
 
         else {
             $objPPT.Quit()
              [System.Runtime.Interopservices.Marshal]::ReleaseComObject($openPresentationWithPasswords) | out-null
              [System.Runtime.Interopservices.Marshal]::ReleaseComObject($objPPT) | out-null
         }
     }
 }
 
 if ($file.Extension -eq ".pdf") {
 
     $arrPasswords | % {
         $passwd = $_
             
         $ghostScriptPath = "C:\Program Files\gs\gs9.14\bin\gswin32.exe"
         
         $savePath = $decrypted_Path+$file.Name
 
         $args = '-q', '-dSAFER', '-dBATCH', '-dNOPAUSE', '-sDEVICE=pdfwrite', '-sFONTPATH=%windir%/fonts;xfonts;.', "-sPDFPassword=$passwd", '-dPDFSETTINGS=/prepress', '-dPassThroughJPEGImages=true', "-sOutputFile=$savePath", $file.FullName
         
         & $ghostScriptPath $args 
         sleep 2
         Get-Process -name gswin32 
 
         if (test-path -path $savePath) {
             move-item $file.fullname -Destination $original_Path -Force
         }
     }
 
     Get-Process -name gswin32 | % {tskill $_.id} | out-null
 }
 $count--
 }
 
 Write-host "`n Complete" -f "Green"
`

