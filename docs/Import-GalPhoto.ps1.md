---
Author: thomas torggler
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3570
Published Date: 2012-08-12t07
Archived Date: 2012-12-21t01
---

# import-galphoto - 

## Description

a function to batch import exchange 2010 gal photos… find the related post on ntsystems.it

## Comments



## Usage



## TODO



## script

`import-galphoto`

## Code

`#
 #
 <#
 .Synopsis
    Import Exchange 2010 Gal photo for one or more users. 
 .DESCRIPTION
    This function invokes the Import-RecipientDataProperty cmdldet to import a picture into the thumbnailPhoto attribute of an Exchange mailbox.
    It assumes you have a directory containing all images to import, with the file names corresponding to the usernames of the mailboxes.
    
    PS X:\> dir *.jpg
 
 
         Directory: c:\temp\pics
 
 
     Mode                LastWriteTime     Length Name
     ----                -------------     ------ ----
     -a---        07.08.2012     19:29       5984 user1.name.jpg
     -a---        16.05.2012     11:58       5984 user2.jpg
     -a---        23.07.2012     15:55       5984 user3.name.jpg
     ...
 
 .EXAMPLE
    PS C:\> Import-GalPhoto -FilePath 'c:\temp\pics'
    This example gets all .jpg pictures from c:\temp\pics and tries to find corresponding Exchange Users. If a user is found, the picture is imported using Import-RecipientDataProperty.
 .EXAMPLE
    PS C:\> dir 'c:\temp\pics' | Select-Object -First 2 | Import-GalPhoto
    This example shows how you can use Get-ChildItem (dir) and do some custom filtering (Where-Object, Select-Object) to get pictures. 
    Then it makes use of pipeline input to import pictures.
 #>
 function Import-GalPhoto
 {
     [CmdletBinding(SupportsShouldProcess=$true, 
                   ConfirmImpact='High')]
     Param
     (
         [Parameter(Mandatory=$true, 
                    ValueFromPipelineByPropertyName=$true,
                    Position=0)]
         [ValidateNotNull()]
         [ValidateNotNullOrEmpty()]
         [Alias('FullName')]
         [IO.FileInfo]
         $FilePath,
 
         [Parameter(Mandatory=$false,
                    ParameterSetName='Parameter Set 1')]
         $LogFile=".\Import-GalPhoto.txt"
     )
 
     Begin
     {
         Write-Verbose 'Execute Begin Block'
         Remove-Item $LogFile -ErrorAction SilentlyContinue
         
         [void][System.Reflection.Assembly]::LoadWithPartialName("System.Drawing")
 
         "$(Get-Date) Function begin" | Add-Content -Path $LogFile -WhatIf:$false
     }
     Process
     {
         Write-Verbose 'Execute Process Block'
 
         [array]$arrPics = Get-ChildItem -Path $FilePath -File -Filter '*.jpg'
         "$(Get-Date) arrPics is $($arrPics)" | Add-Content -Path $LogFile -WhatIf:$false
 
         $arrPics | ForEach-Object {
             [IO.FileInfo]$objPicture = $_.FullName
             
             [string]$strUserName = $_.BaseName
             
             Write-Verbose "`$objPicture is $objPicture"
             Write-Verbose "`$strUserName is $strUserName"
             "$(Get-Date) objPicture is $($objPicture)" | Add-Content -Path $LogFile -WhatIf:$false
             "$(Get-Date) strUserName is $($strUserName)" | Add-Content -Path $LogFile -WhatIf:$false
     
             try {
                 Write-Verbose "Try to find User $strUserName"
                 Start-Sleep -Seconds 1
                 $exchangeUser = Get-User -Identity $strUserName -ErrorAction Stop
                 $Continue = $true
 
                 $objImage = [System.Drawing.Image]::FromFile($objPicture)
 
                 $objFileData = ([Byte[]]$(Get-Content -Path $objPicture -Encoding Byte -ReadCount 0 -ErrorAction Stop))
                     
                 Write-Verbose "`$exchangeUser is $($exchangeUser.SamAccountName)"
                 Write-Verbose "`$objImage PhysicalDimensionis is $($objImage.PhysicalDimension)"
                 Write-Verbose "`$objPicture filesize(KB) is $($objPicture.Length / 1kb)"
                 "$(Get-Date) exchangeUser is $($exchangeUser.SamAccountName)" | Add-Content -Path $LogFile -WhatIf:$false
                 "$(Get-Date) Image PhysicalDimension is $($objImage.PhysicalDimension)" | Add-Content -Path $LogFile -WhatIf:$false
                 "$(Get-Date) Image filesize(KB) is $($objPicture.Length / 1kb)" | Add-Content -Path $LogFile -WhatIf:$false
 
             } catch {
                 $Continue = $false
 
                 Write-Warning "Could not find User $strUserName"
                 "$(Get-Date) Could not find $($strUserName)" | Add-Content -Path $LogFile -WhatIf:$false
             }
             
             if ($Continue -and (($objImage.Height -le 96) -and ($objImage.Width -le 96) -and ($objPicture.Length -le 10kb))){
                 $Continue = $true
 
                 $objImage.Dispose()                
 
                 Write-Verbose "Picture $($objPicture) is ok"
             } else {
                 $Continue = $false
                 
                 Write-Verbose "Picture $($objPicture) is not ok"
                 "$(Get-Date) Skipped $($objPicture) size not ok" | Add-Content -Path $LogFile -WhatIf:$false
             }
                     
             if ($Continue -and ($pscmdlet.ShouldProcess("$($exchangeUser.SamAccountName)", "Importing Recipient Data Property from file $objPicture"))) {
                 
                 Import-RecipientDataProperty -Identity $exchangeUser.Identity -Picture -FileData $objFileData    
             }
         }
     }
     End
     {
         Write-Verbose 'Execute End Block'
         
         "$(Get-Date) Function end" | Add-Content -Path $LogFile -WhatIf:$false
     }
 }
`

