---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 1.00
Encoding: ascii
License: cc0
PoshCode ID: 4511
Published Date: 2015-10-08t08
Archived Date: 2015-10-29t00
---

# convertto-adthumbnail - 

## Description

converts pictures to byte arrays and saves a copy of the new file on disk.

## Comments



## Usage



## TODO



## function

`convertto-adthumbnail`

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 function ConvertTo-ADThumbnail
 {
     <#
     .SYNOPSIS
     This script cmdlet converts a image file for use in Active Directory.
 
     .DESCRIPTION
     It converts a file to a byte variable that can be written to the Active Directory thumbnail attribute. The size is also change to < 9 kb (default).
 
     .PARAMETER PictureFile
     Provide the path to the picture file here.
 
     .PARAMETER PictureSize
     Set the picture size in KB here. Default is 9 kb. (to work with Office 365)
 
     .PARAMETER OutputDir
     Specify the folder where you want the copy of the original file (but now smaller) stored.
 
     .EXAMPLE
     ConvertTo-ADThumbnail -PictureFile '.\MyPicture.jpg'
 
     .EXAMPLE
     ConvertTo-ADThumbnail -PictureFile '.\MyPicture.jpg' -PictureSize 100
 
     #>
 
     [CmdletBinding()]
     param(
       [Parameter(Mandatory=$true, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
       [Alias('Fullname')]
       [string] $PictureFile,
       [Parameter(Mandatory=$false)]
       [string] $OutputDir = $(Get-Location | select -ExpandProperty Path),
       [Parameter(Mandatory=$false)]
       [Alias('Size')]
       [int] $PictureSize = 9)
 
     BEGIN {
         [reflection.assembly]::LoadWithPartialName("System.Drawing") | Out-Null
         $SupportedExtensions=@()
         $SupportedExtensions=".jpg",".gif",".bmp",".png"
     }
 
     PROCESS {
 
         $PictureFileObj=Get-ChildItem $PictureFile
 
         if ($PictureFileObj.Extension -in $SupportedExtensions) {
 
             $OriginalPicture = New-Object System.Drawing.Bitmap $PictureFile
 
             $PictureFileSize=($PictureFileObj | select -ExpandProperty Length)/1KB
 
             $NewPictureName="$($PictureFileObj.BaseName)-ADThumbnail.jpg"
 
             if ($OutputDir -like "`.\*") {
                 $CurrentDirectory=$(Get-Location | select -ExpandProperty Path)
                 $PathToResolve="$CurrentDirectory$OutputDir"
                 $OutputPath=[System.IO.Path]::GetFullPath($PathToResolve)
             }
             else {
                 $OutputPath=[System.IO.Path]::GetFullPath($OutputDir)
             }
             
 
             $OutFile="$OutputPath\$NewPictureName"
 
             if ($PictureFileSize -gt $PictureSize) {
 
                 $NewPictureFileSize=$PictureFileSize
             
                 [decimal] $ShrinkFactorImage=1.00
                 $ShrinkNr=0
 
                 while ($NewPictureFileSize -gt $PictureSize) {
                     
                     $ShrinkNr++
                     [decimal] $ShrinkFactorImage=(1.00-($ShrinkNr*0.01))
 
                     Remove-Item $OutFile -Force -ErrorAction SilentlyContinue
 
                     [int] $NewPictureWidth=$OriginalPicture.Width*$ShrinkFactorImage
                     [int] $NewPictureHeight=$OriginalPicture.Height*$ShrinkFactorImage
 
                     $NewPicture=New-Object System.Drawing.Bitmap $NewPictureWidth,$NewPictureHeight
 
                     $NewPictureDrawing=[System.Drawing.Graphics]::FromImage($NewPicture)
                     $NewPictureDrawing.InterpolationMode = [System.Drawing.Drawing2D.InterpolationMode]::HighQualityBicubic
 
                     $NewPictureDrawing.DrawImage($OriginalPicture, 0, 0, $NewPictureWidth, $NewPictureHeight)
 
 
                     $FileExist=Test-Path $OutFile
 
                     if ($FileExist) {
                         Remove-Item $OutFile -Force -ErrorAction SilentlyContinue
                     }
 
                     try {
                         $NewPicture.Save($OutFile,([system.drawing.imaging.imageformat]::jpeg))
                     }
                     catch {
                     }
 
                     $NewPicture.Dispose()
                     $NewPictureDrawing.Dispose()
 
                     try {
                         $NewPictureFileSize=(Get-ChildItem $OutFile -ErrorAction Stop | select -ExpandProperty Length)/1KB
                     }
                     catch {
                         return
                     }
                 }
 
                 $ByteArray = [byte[]](Get-Content $OutFile -Encoding byte)
                 
             }
             else {
                 $ByteArray = [byte[]](Get-Content $PictureFile -Encoding byte)
                 Copy-Item $PictureFile $OutFile -Force
 
                 $NewPictureFileSize=$PictureFileSize
                 $NewPictureWidth=$OriginalPicture.Width
                 $NewPictureHeight=$OriginalPicture.Height
             }
 
             $returnObject = New-Object System.Object
             $returnObject | Add-Member -Type NoteProperty -Name OrgFilename -Value $PictureFile
             $returnObject | Add-Member -Type NoteProperty -Name OrgFileSize -Value $PictureFileSize
             $returnObject | Add-Member -Type NoteProperty -Name OrgFileWidth -Value $OriginalPicture.Width
             $returnObject | Add-Member -Type NoteProperty -Name OrgFileHeight -Value $OriginalPicture.Height
             $returnObject | Add-Member -Type NoteProperty -Name NewFilename -Value $OutFile
             $returnObject | Add-Member -Type NoteProperty -Name NewFileSize -Value $NewPictureFileSize
             $returnObject | Add-Member -Type NoteProperty -Name NewFileWidth -Value $NewPictureWidth
             $returnObject | Add-Member -Type NoteProperty -Name NewFileHeight -Value $NewPictureHeight
             $returnObject | Add-Member -Type NoteProperty -Name ThumbnailByteArray -Value $ByteArray
 
             Write-Output $returnObject
 
             $OriginalPicture.Dispose()
             $ByteArray=$null
 
 
         }
     }
     END {
     }
 }
`

