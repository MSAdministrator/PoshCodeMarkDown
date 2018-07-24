---
Author: robertthebruce
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6003
Published Date: 2016-09-07t08
Archived Date: 2016-05-17t13
---

# join-textfile - 

## Description

simple script to join multiple text files into one long string with delimiters ready to be split out again. split-textfile scrip still under development!

## Comments



## Usage



## TODO



## script

`join-textfile`

## Code

`#
 #
 <#
 .Synopsis
    Joins a list of text files into 1 large text file.
 .DESCRIPTION
    Long description
 .EXAMPLE
    Example of how to use this cmdlet
 .EXAMPLE
    Another example of how to use this cmdlet
 #>
 function Join-TextFile
 {
     [CmdletBinding()]
     [OutputType([string])]
     Param
     (
         [Parameter(Mandatory=$true,
                    ValueFromPipeline=$true,
                    Position=0)]
         [System.IO.FileInfo[]]
         $TextFiles,
 
         [string]
         $Separator = (Get-Date -Format s)
     )
 
     Begin
     {
         
         Function Get-SHA256Hash($TextToHash)
         {
             $SHA256 = New-Object System.Security.Cryptography.SHA256Managed
             $ByteArray = [System.Text.Encoding]::UTF8.GetBytes($TextToHash)
             $HashByteArray = $SHA256.ComputeHash($ByteArray)
     
             Return [System.Convert]::ToBase64String($HashByteArray)
         }
         
 
         $SessionID = Get-SHA256Hash ((gwmi win32_OperatingSystem|Select *) -join ";").ToString()
 
         $HeaderText = "##:Join-TextFile Start Seperator=#/$Separator/#;SessionID=$SessionID"
         $FooterText = "##:Join-TextFile End Seperator=#/$Separator/#;SessionID=$SessionID"
 
         Write-Output $HeaderText
 
 
 
 
     
     Process
     {
         ForEach ($File in $TextFiles)
         {
             Write-Verbose $File
             $FileContents = Get-Content $File -Raw -Encoding UTF8
             $ContentsHash = Get-SHA256Hash $FileContents
             Write-Output ($SeparatorStart -replace '#\/\<Filename\>\/#',"#/$($File.Name)/#" -replace '#\/\<ContentsHash\>\/#',"#/$ContentsHash/#")
             Write-Output $FileContents
             Write-Output ($SeparatorEnd -replace '#\/\<Filename\>\/#$',"#/$($File.Name)/#")
         }
     
     End
     {
         Write-Output $FooterText
 }
`

