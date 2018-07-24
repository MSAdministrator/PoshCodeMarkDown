---
Author: rcookiemonster
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5614
Published Date: 2015-11-29t01
Archived Date: 2015-03-23t14
---

# copy-coloredhtml - 

## Description

html functions from the powershell pack.

## Comments



## Usage



## TODO



## function

`write-colorizedhtml`

## Code

`#
 #
 function Write-ColorizedHTML { 
         <#
         .Synopsis
             Writes Windows PowerShell as colorized HTML
         .Description
             Outputs a Windows PowerShell script as colorized HTML.
             The script is wrapped in <PRE> tags with <SPAN> tags defining color regions.
         .Example
             Write-ColoredHTML {Get-Process}
         .NOTES
             From the PowerShell Pack
         #>
         param(
             [Parameter(Mandatory=$true)]
             [String]$Text,
             [Int]$Start = -1,
             [Int]$End = -1)
     
         trap { break } 
         #
         #
         $parse_errs = $null
         $tokens = [Management.Automation.PsParser]::Tokenize($text,
             [ref] $parse_errs)
  
         if ($parse_errs) {
             $parse_errs
             return
         }
         $stringBuilder = New-Object Text.StringBuilder
         $null = $stringBuilder.Append("<pre class='PowerShellColorizedScript'>")
         $lastToken = $null
         foreach ($t in $tokens)
         {
             if ($lastToken) {
                 $spaces = " " * ($t.Start - ($lastToken.Start + $lastToken.Length))
                 $null = $stringBuilder.Append($spaces)
             }
             if ($t.Type -eq "NewLine") {
                 $null = $stringBuilder.Append("            
 ")
             } else {
                 $chunk = $text.SubString($t.start, $t.length)
                 $color = $psise.Options.TokenColors[$t.Type]            
                 $redChunk = "{0:x2}" -f $color.R
                 $greenChunk = "{0:x2}" -f $color.G
                 $blueChunk = "{0:x2}" -f $color.B
                 $colorChunk = "#$redChunk$greenChunk$blueChunk"
                 $null = $stringBuilder.Append("<span style='color:$colorChunk'>$chunk</span>")                    
             }                       
             $lastToken = $t
         }
         $null = $stringBuilder.Append("</pre>")
         $stringBuilder.ToString()
      }
 
 function Copy-ColoredHTML { 
     <#
     .Synopsis
         Copies the currently selected text in the current file as colorized HTML
     .Description
         Copies the currently selected text in the current file as colorized HTML
         This allows for a user to paste colorized scripts into web pages or blogging 
         software
     .Example
         Copy-ColoredHTML
     .NOTES
         From the PowerShell Pack
     #>
     [cmdletbinding()]
     param()
     
     if (-not $psise.CurrentFile)
     {
         Write-Error 'You must have an open script file'
         return
     }
 
     
 
     $selectedRunspace = $psise.CurrentFile
     $selectedEditor=$selectedRunspace.Editor
 
     $FullText = $selectedEditor.Text -replace '\r\n', "`n"
     if (-not $selectedEditor.SelectedText)
     {
         $colorizedText = Write-ColorizedHTML $selectedEditor.Text
     }
     else
     {
         $colorizedText = Write-ColorizedHTML $selectedEditor.SelectedText
     }
 
     [Windows.Clipboard]::SetText($colorizedText)
  
  }
`

