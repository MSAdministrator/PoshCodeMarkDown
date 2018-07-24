---
Author: archdeacon
Publisher: 
Copyright: 
Email: 
Version: 2.1
Encoding: ascii
License: cc0
PoshCode ID: 3123
Published Date: 2012-12-24t03
Archived Date: 2012-01-14t07
---

# set logfile length - 

## Description

set any text file to a fixed number of lines. useful for maintaining files such as the ps transcript log. now option added to remove blank lines.

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
 Set any text file to a fixed number of lines. Use 'Get-Help .\SetFileLines
 -full' to view Help for this file.
 .DESCRIPTION
 This script will maintain the PS Transcript file (default setting), or any 
 text file, at a fixed length, ie matching the number of lines entered.
 However, omitting the lines parameter will just remove any blank lines; and 
 using the -Blanks switch will remove blanks from the desired length. Can be 
 included in $profile.
 .EXAMPLE
 Set-FileLines -File c:\Scripts\anyfile.txt
 Remove all blank lines from the file 'anyfile.txt'.
 .EXAMPLE  
 Set-FileLines 1500 -Blanks
 This will set the file length of 'Transcript.txt' to 1500 lines and also 
 remove all blank lines.
 .EXAMPLE
 Set-FileLines
 Remove any blank lines from the default file 'Transcript.txt'.
 .NOTES
 The 'Lines' property returned by '(Get-Content $file | Measure-Object -line)'
 excludes any blank lines so the resulting calculated totals may not be 100% 
 accurate.
 The author can be contacted via www.SeaStarDevelopment.Bravehost.com
 V2.1 Use 'Switch -regex' instead of 'Get-Content | foreach', 22 Dec 2011. 
 #>
 
 
 Param ([int]   $lines = 0,
        [String]$file = "$pwd\Transcript.txt",
        [Switch]$blanks)
 
 if ($file -notlike "*.txt") {
     [System.Media.SystemSounds]::Hand.Play()
     Write-Warning "This script can only process .txt files"
     exit 1
 }
 if (!(Test-Path $file)) {
     [System.Media.SystemSounds]::Hand.Play()
     Write-Warning "File $file does not exist - please enter valid filename."
     exit 1
 }
 
 [int]$count = 0
 [int]$blankLines = 0
 $encoding = 'Default'
 $errorActionPreference = 'SilentlyContinue'
     [int]$extra = 1
     [int]$count = 1
 }
 else {
     $fileLength = (Get-Content $file | Measure-Object -line)
 if ($extra -gt 0) {
     $fileLength = $null
     $date = "{0:G}" -f [DateTime]::Now
     Write-Output "$date Starting maintenance on file <$file>"
     $tempfile = [IO.Path]::GetTempFileName()
     if ($file -like "*transcript*.txt") {
         $encoding = 'Unicode' 
         Stop-Transcript | Out-Null
     }
     switch -regex -file $file {
       { $count -lt $extra } { $count++; continue}
       '^\s*$'               { if ($blanks) {
                                  $blankLines++
                                  continue  
                               }
                               $_ | Out-File $tempFile -encoding $encoding -Append -Force  
       default               { $_ | Out-File $tempFile -encoding $encoding -Append -Force } 
     if (!$?) {
         [System.Media.SystemSounds]::Hand.Play()
         Write-Warning "$($error[0]) Application terminating." 
         Remove-Item $tempfile
         $ErrorActionPreference = 'Continue'
         exit 2
     }
     Move-Item $tempfile -Destination $file -Force 
         $tag = "$blankLines blank lines removed."
     }
     elseIf ($blanks) {
         $tag = "$extra lines removed (+ $blankLines blank)."
     }
     else {
         $tag = "$extra lines removed."
     }
     if (($file -like "*transcript.txt") -and $status) {
         Start-Transcript -append -path $file -force | Out-Null
     }
     Write-Output "Maintenance of file completed: $tag"
 }
 else {
     Write-Output "[$file] Filesize ($($FileLength.Lines) lines) is below minimum; no lines removed."
 }
 $ErrorActionPreference = 'Continue'
`

