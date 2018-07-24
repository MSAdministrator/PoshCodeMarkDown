---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3314
Published Date: 
Archived Date: 2016-05-19t03
---

#  - 

## Description

replacement of write-host function to work around an issue where use of write-host can cause an eventual problem with launching exes from within the same powershell session. see https

## Comments



## Usage



## TODO



## function

`write-host`

## Code

`#
 #
 function Write-Host
 {
     <#
     .SYNOPSIS
         Replacement of Write-Host function to work around an issue where use of Write-Host can
         cause an eventual problem with launching EXEs from within the same Powershell session.
 
     .DESCRIPTION
         This Write-Host replacement can act as a temporary workaround to an issue with some 
         PowerShell versions currently released. The high-level description of the problem 
         is that excessive use of Write-Host eventually leads to failures to execute EXEs
         from the same PowerShell session. 
 
         For platforms where the problem exists, the following has been seen to reproduce the issue:
         for( $i = 0; $i -lt 1000; $i += 1 ) { Write-Host line }; whoami.exe; "0x{0:X}" -f $LASTEXITCODE
 
         On repro, the $LASTEXITCODE will be 0xC0000142. 
 
         This issue is described by the following Connect article:
         https://connect.microsoft.com/PowerShell/feedback/details/496326/stability-problem-any-application-run-fails-with-lastexitcode-1073741502
 
         Some folks workaround the issue by simply using pipeline output to display status messages. 
         Use the pipeline in this manner may be ideal for situations where it is not used for other
         purposes.
         
         If you have scripts that use the pipeline for other purposes, and rely on Write-Host 
         for non-pipeline status output, where your scripts hit this issue, perhaps this workaround
         may be of use. 
 
     .PARAMETER Object
         Object to be outputed to stdout.
 
     .PARAMETER NoNewLine
         Specify this flag if you don't what the output to end with NewLine character
 
     .PARAMETER ForegroundColor
         Specifies the text color. There is no default.
         Use Get-Colors command to output all available colors in color.
 
         Possible color names taken from '[ConsoleColor] | gm -Static' are:
             Blue    | DarkBlue
             Cyan    | DarkCyan
             Gray    | DarkGray
             Green   | DarkGreen
             Magenta | DarkMagenta
             Red     | DarkRed
             Yellow  | DarkYellow
             White   | Black
 
     .PARAMETER BackgroundColor
         Specifies the background color. There is no default.
         Use Get-Colors command to output all available colors in color.
 
         Possible color names taken from '[ConsoleColor] | gm -Static' are:
             Blue    | DarkBlue
             Cyan    | DarkCyan
             Gray    | DarkGray
             Green   | DarkGreen
             Magenta | DarkMagenta
             Red     | DarkRed
             Yellow  | DarkYellow
             White   | Black
 
     .EXAMPLE
         Write-Console "=)", "test" -ForegroundColor Green -NoNewLine
 
         Prints all items in passed string array to stdout with green color used in console.
 
     .LINK
         Write-Colorized
         Write-Host
         Get-Colors
     #>
 
     param
     (
         [Parameter(Mandatory = $true, Position = 0, ValueFromRemainingArguments = $true, ValueFromPipeline = $true)]
         [object] $Object,
         [object] $Separator = " ",
         [ConsoleColor] $ForegroundColor,
         [ConsoleColor] $BackgroundColor,
         [switch] $NoNewLine
     )
 
     begin
     {
         function printObject($o)
         {
             function WriteObject([string]$s)
             {
                 if ($s.Length -gt 0)
                 {
                     if ($consoleHost) {
                         [Console]::Write($s);
                     } else {
                         $host.UI.Write($ForegroundColor, $BackgroundColor, $s)
                     }
                 }
             }
 
             if ($o -eq $null) {
                 return;
             }
 
             if ($o -is [string]) {
                 WriteObject $o
             } else {
 
                 if ($o -is [System.Collections.IEnumerable]) {
                     $objectEnumerator = $o.GetEnumerator();
                     $printSeparator = $false;
                     foreach ($element in $objectEnumerator) {
 
                         if ($Separator -ne $null -and $printSeparator) {
                             WriteObject $Separator
                         } 
 
                         printObject $element
 
                         $printSeparator = $true;
                     }
                 } else {
                 
                     WriteObject $o.ToString()
                 }
             }
         }
 
         $consoleHost = $host.Name -eq "ConsoleHost"
         if (!$consoleHost)
         {
             if (!$ForegroundColor)
             {
                 $ForegroundColor = $host.UI.RawUI.ForegroundColor
             }
             if (!$BackgroundColor)
             {
                 $BackgroundColor = $host.ui.RawUI.BackgroundColor
             }
         }
     }
 
     process
     {
         if ($consoleHost)
         {
             if( $ForegroundColor )
             {
                 $previousForegroundColor = [Console]::ForegroundColor
                 [Console]::ForegroundColor = $ForegroundColor
             }
 
             if( $BackgroundColor )
             {
                 $previousBackgroundColor = [Console]::BackgroundColor
                 [Console]::BackgroundColor = $BackgroundColor
             }
         }
 
         try
         {
             printObject $Object $consoleHost
 
             if( $NoNewLine -eq $false)
             {
             }
 
         }
         finally
         {
             if ($consoleHost)
             {
                 if( $ForegroundColor )
                 {
                     [Console]::ForegroundColor = $previousForegroundColor
                 }
 
                 if( $BackgroundColor )
                 {
                     [Console]::BackgroundColor = $previousBackgroundColor
                 }
             }
         }
     }
 }
 
 function Test-WriteConsole {
 
     Write-Host  'new console write';
     Write-Host  @(1,2,3,4) -Separator '-sep-'
     Write-Host  @(1,2,3,4) -Separator '-sep-' -ForegroundColor Yellow
     Write-Host  @(1,2,3,4) -Separator '-sep-' -ForegroundColor Black -BackgroundColor White
     Write-Host  ([int32]123) -Separator '-sep-' -ForegroundColor Cyan -BackgroundColor DarkGreen
     Write-Host  '123' -Separator '-sep-' -ForegroundColor Green -BackgroundColor Black
     Write-Host  @([int32]123,[int32]123,[int32]123,[int32]123,[int32]123) -Separator '-sep-' -ForegroundColor Cyan -BackgroundColor DarkGreen
     Write-Host 'This is one write. ' -NoNewLine;
     Write-Host 'This is another write on the same line. ' -NoNewLine -ForegroundColor Black -BackgroundColor White
     Write-Host @('abc', [uint32]123, 'def', ' array on same line. ') -Separator ',' -NoNewLine -ForegroundColor Cyan -BackgroundColor DarkMagenta
     Write-Host 'This is the last sentence on this line.'
     Write-Host 'This should be on a new line.';
     1..5 | Write-Host
     Write-Host 1 2 3 4 5
 } 
 
 Test-WriteConsole
`

