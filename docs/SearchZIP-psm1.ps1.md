---
Author: archdeacon
Publisher: 
Copyright: 
Email: 
Version: 2.4
Encoding: ascii
License: cc0
PoshCode ID: 3248
Published Date: 2012-02-21t04
Archived Date: 2012-02-23t04
---

# searchzip.psm1 - 

## Description

in any directory containing a large number of zip/rar compressed web page files this procedure will search each individual file name for simple text strings, listing both the source rar/zip file and the individual file name containing the string.

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
 Search for (filename) strings inside compressed ZIP or RAR files (V2.4).
 .DESCRIPTION
 In any directory containing a large number of ZIP/RAR compressed Web Page files 
 this procedure will search each individual file name for simple text strings, 
 listing both the source RAR/ZIP file and the individual file name containing
 the string. The relevant RAR/ZIP can then be subsequently opened in the usual
 way.
 Using the '-Table' (or alias '-GridView') switch will show the results with the 
 Out-GridView display.
 .EXAMPLE
 extract -find 'library' -path d:\scripts
 
 Searching for 'library' - please wait... (Use CTRL+C to quit)
 [Editor.zip] File: Windows 7 Library Procedures.mht
 [Editor.rar] File: My Library collection from 1974 Idi Amin.html
 [Test2.rar] File: Playlists from library - Windows 7 Forums.mht
 [Test3.rar] File: Module library functions UserGroup.pdf
 Folder 'D:\Scripts' contains 4 matches for 'library' in 4 file(s).
 .EXAMPLE
 extract -find 'backup' -path desktop
 
 Searching for 'backup' - please wait... (Use CTRL+C to quit)
 [Web.zip] File: How To Backup User and System Files.mht
 [Pages.rar] File: Create an Image Backup.pdf
 Folder 'D:\Scripts' contains 2 matches for 'backup' in 2 file(s).
 .EXAMPLE
 extract pdf desk
 
 Searching for 'pdf' - please wait... (Use CTRL+C to quit)
 [Test1.rar] File: VirtualBox_ Guest Additions package.pdf
 [Test2.rar] File: W8 How to Install Windows 8 in VirtualBox.pdf
 [Test2.rar] File: W8 Install Windows 8 As a VM with VirtualBox.pdf
 Folder 'C:\Users\Sam\desktop' contains 3 matches for 'pdf' in 2 file(s).
 
 This example uses the 'extract' alias to find all 'pdf' files on the desktop.
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
          [switch][alias("GRIDVIEW")]$table)
    Set-StrictMode -Version 2
    [int]$count = 0
    if ($path -like 'desk*') {
       $path = Join-Path $home 'desktop\*'
    }
    else {
       $xpath = Join-Path $path '*' -ea 0
       if (!($?) -or !(Test-Path $path)) {
          Write-Warning "Path '$path' is invalid - resubmit"
          return
       }
       $path = $xpath
    }
 
    Get-ChildItem $path -include '*.rar','*.zip' |
       Select-String -SimpleMatch -Pattern $find |
          foreach-Object `
             -begin {
                 $container = @{}
                 $folder = $path.Replace('*','')
                 $lines = @{}
                 $regex = '(?s)^(?<zip>.+?\.(?:zip|rar)):(?:\d+):.*\\(?<file>.*\.(mht|html?|pdf))(.*)$'
                 Write-Host "Searching for '$find' - please wait... (Use CTRL+C to quit)" 
             } `
             -process {
                 $_ = $_ -replace '/','\'
                 if ( $_ -match $regex ) {
                    if ($container.keys -contains $matches.zip) {
                    }
                    else {
                       $container[$matches.zip] = 1
                    }
                    $source = $matches.zip -replace [regex]::Escape($folder)
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
                 $total = "in $($container.count) file(s)." 
                 $title = "Folder '$($path.Replace('\*',''))' contains $($count) matches for '$find' $total"
                 if ($table -and $count -gt 0) {        
                    $lines.GetEnumerator() | 
                       Select-Object @{name = 'Source';expression = {$_.Key.SubString(5)}},
                                     @{name = 'File'  ;expression = {$_.Value}} |
                          Sort-Object File |
                             Out-GridView -Title $title
                 }
                 else {
                    if ($count -eq 0) {
                       $title = "Folder '$($path.Replace('\*',''))' contains no matches for '$find'."   
                    }
                    Write-Host $title
                 } 
             }
 
 New-Alias extract SearchZIPfiles    -Description 'Find Web files inside ZIP/RAR'
 Export-ModuleMember -Function SearchZIPfiles -Alias Extract
`

