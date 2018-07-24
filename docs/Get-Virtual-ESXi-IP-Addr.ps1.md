---
Author: carter shanklin
Publisher: 
Copyright: 
Email: 
Version: 4.0
Encoding: ascii
License: cc0
PoshCode ID: 1823
Published Date: 
Archived Date: 2010-05-11t02
---

# get virtual esxi ip addr - 

## Description

this code determines the ip address of a virtual esxi vm in what might just be the dumbest way possible, namely by taking a screenshot of it, ocring the results and extracting the ip. for educational purposes only. unfortunately it’s pretty slow due to some cmdlet slowness. requires powercli 4.0 u1, powershell v2, and microsoft office document imaging (modi) to be installed and configured.

## Comments



## Usage



## TODO



## function

`get-virtualesxiip`

## Code

`#
 #
 function Get-VirtualEsxiIp {
 	param($vm)
 	$tmpFileTemplate = ($env:TEMP + "\ipdetect-")
 	$tmpFile = $tmpFileTemplate + (Get-Random) + ".png"
 
 	$view = $vm | Get-View -Property Name
 	$path = $view.CreateScreenShot()
 	$path -match "([^/]+/[^/]+$)" | Out-Null
 	$relativePath = $matches[1]
 
 	$datastore = $vm | Get-Datastore
 	$remotePath = ($datastore.DatastoreBrowserPath + "\" + $relativePath)
 	Write-Verbose ("Use remote path " + $remotePath)
 
 	Write-Verbose ($remotePath + ", " + $tmpFile)
 	Copy-DatastoreItem $remotePath $tmpFile
 
 	$output = Apply-Ocr -pngFile $tmpFile
 
 	$output -match "http://([^/]+)" | Out-Null
 	$ip = $matches[1]
 
 	Write-Verbose "Cleaning up"
 	Remove-Item -Force $tmpFile
 
 	Write-Output $ip
 }
 
 function Apply-Ocr {
 	param($pngFile)
 	$tmpFileTemplate = ($env:TEMP + "\ipdetectocr-")
 	$tmpFile = $tmpFileTemplate + (Get-Random) + ".tif"
 
 	[reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 	$image = [System.Drawing.Image]::FromFile($pngFile)
 	$image.Save($tmpFile, [System.Drawing.Imaging.ImageFormat]::TIFF)
 
 	$modi = new-object -com MODI.Document
 	$modi.Create($tmpFile)
 	$modi.OCR()
 	$OCRText = ($modi.Images | select -expand Layout).Text
 	$modi.Close()
 	$image.Dispose()
 
 	foreach ($file in (
 		Get-Childitem $tmpFileTemplate* |
 		    Where { $_.LastWriteTime -lt ((Get-Date).addMinutes(-1)) } )) {
 		Remove-Item -Force $file
 	}
 
 	Write-Output $OCRText
 }
 
`

