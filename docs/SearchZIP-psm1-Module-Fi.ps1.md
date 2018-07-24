---
Author: archdeacon
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3236
Published Date: 2012-02-14t20
Archived Date: 2012-02-18t15
---

# searchzip.psm1 module fi - 

## Description

if there are a large number of, for example, saved web pages (mht,htm,html,pdf)

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function SearchZIPfiles {
 <#
 .SYNOPSIS 
 Search for (filename) strings inside compressed ZIP or RAR files.
 .DESCRIPTION
 In any directory containing a large number of ZIP/RAR compressed Web Page files 
 this procedure will search each individual file name for simple text strings, 
 listing both the source RAR/ZIP file and the individual file name containing
 the string. The relevant RAR/ZIP can then be subsequently opened in the usual
 way.
 Using the '-Table' switch will show the results with the Out-GridView display.
 .EXAMPLE
 extract -find 'library' -path d:\scripts
 
 Searching for 'library' - please wait... (Use CTRL+C to quit)
 [Editor.zip] File: Windows 7 Library Procedures.mht
 [Editor.rar] File: My Library collection from 1974 Idi Amin.html
 [Test2.rar] File: Playlists from library - Windows 7 Forums.mht
 [Test3.rar] File: Module library functions UserGroup.pdf
 A total of 4 matches for 'library' found in folder 'D:\Scripts'.
 .EXAMPLE
 extract -find 'backup' -path desktop
 
 Searching for 'backup' - please wait... (Use CTRL+C to quit)
 [Web.zip] File: How To Backup User and System Files.mht
 [Pages.rar] File: Create an Image Backup.pdf
 A total of 2 matches for 'backup' found in folder 'C:\Users\Sam\Desktop'.
 .NOTES
 The first step will find any lines containing the selected pattern (which can
 be anywhere in the line). Each of these lines will then be split into 2 
 headings: Source and Filename.
 Note that there may be the odd occasion where a 'non-readable' character in the
 returned string slips through the net! 
 .LINK
 Web Address Http://www.SeaStarDevelopment.BraveHost.com
 #>
    [CmdletBinding()]
    param([string][string][Parameter(Mandatory=$true)]$Find,
          [string][ValidateNotNullOrEmpty()]$path = $pwd,
          [switch]$table)
 
    [int]$count = 0
    if ($path -like 'desk*') {
       $path = Join-Path $home 'desktop\*'
    }
    else {
       $path = Join-Path $path '*'
    }
    if (!(Test-path $path)) {
       Write-Warning "Path '$($path.Replace('\*',''))' is invalid - resubmit"
       return
    }
    $folder = $path.Replace('*','')
    $lines = @{}
    $regex = '(?s)^(?<zip>.+?\.(?:zip|rar)):(?:\d+):.*\\(?<file>.*\.(mht|html?|pdf))(.*)$'
 
    Get-ChildItem $path -include '*.rar','*.zip' |
       Select-String -SimpleMatch -Pattern $find |
          foreach-Object `
             -begin { Write-Host "Searching for '$find' - please wait... (Use CTRL+C to quit)" } `
             -process {
                 $_ = $_ -replace '/','\'
                 if ( $_ -match $regex ) {
                    $source = $matches.zip.Replace($folder,'')
                    $file   = $matches.file
                    if ($table) {
                       $key = "{0:D4}" -f $count
                    }
                    else {
                       Write-Host "[$source] File: $file"
                    }
                    $count++ 
                 }
             } `
             -end { 
                 $title = "A total of $($count) matches for '$find' found in host folder '$($path.Replace('\*',''))'."
                 if ($table -and ($lines.count -gt 0)) {        
                    $lines.GetEnumerator() | 
                       Select-Object @{name = 'Source';expression = {$_.Key.SubString(5)}},
                                     @{name = 'File'  ;expression = {$_.Value}} |
                          Sort-Object File |
                             Out-GridView -Title $title
                 }
                 else {
                    Write-Host $title
                 } 
             }
 
 New-Alias extract SearchZIPfiles    -Description 'Find Web files inside ZIP/RAR'
 Export-ModuleMember -Function SearchZIPfiles -Alias Extract
`

