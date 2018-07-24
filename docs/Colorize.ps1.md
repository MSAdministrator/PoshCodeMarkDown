---
Author: anonymous
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4262
Published Date: 2013-06-26t14
Archived Date: 2013-06-29t06
---

# colorize.ps1 - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage



## TODO



## function

`copy-colored`

## Code

`#
 #
 function Copy-Colored
 {
     <#
     .Synopsis
         Copies the currently selected text in the current file with colorization
     .Description
         Copies the currently selected text in the current file with colorization.
         This allows for a user to paste colorized scripts into Word or Outlook
     .Example
         Copy-Colored  
     #>
     param()
     
     function Colorize
     {
 
         param([string]$Text, [int]$Start = -1, [int]$End = -1, [int]$FontSize = 12)
         trap { break }
         $rtb = New-Object Windows.Forms.RichTextBox    
         $rtb.Font = New-Object Drawing.Font "Consolas", $FontSize 
         $rtb.Text = $Text
 
         $parse_errs = $null
         $tokens = [system.management.automation.psparser]::Tokenize($rtb.Text,
             [ref] $parse_errs)
 
         if ($parse_errs) {
             $parse_errs
             return
         }
         $ColorPalette = New-ScriptPalette 
 
         foreach ($t in $tokens) {
             $rtb.Select($t.start, $t.length)
             $color = $ColorPalette[$t.Type.ToString()]            
             if ($color) {
                 $rtb.selectioncolor = [drawing.color]::FromArgb($color.A, 
                     $color.R, 
                     $color.G, 
                     $color.B)
             }
         }
         if ($start -eq -1 -and $end -eq -1) {
             $rtb.select(0,$rtb.Text.Length)
         } else {
             $rtb.select($start, $end)
         }
         $rtb.Copy()
     }
     
     $text = Get-CurrentOpenedFileText    
     $selectedText = Select-CurrentText -NotInOutput -NotInCommandPane
            
     if (-not $selectedText) {
         $TextToColor = ($Text -replace '\r\n', "`n")
     } else {        
         $TextToColor = ($selectedText -replace '\r\n', "`n")
     }
     Colorize $TextToColor  
 }
 
 function Write-ColorizedHTML {
     <#
     .Synopsis
         Writes Windows PowerShell as colorized HTML
     .Description
         Outputs a Windows PowerShell script as colorized HTML.
         The script is wrapped in <PRE> tags with <SPAN> tags defining color regions.
     .Example
         Write-ColoredHTML {Get-Process}
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
             
             $colorChunk = "#$redChunk$greenChunk$blueChunk"
             $null = $stringBuilder.Append("<span style='color:$colorChunk'>$chunk</span>")                    
         }                       
         $lastToken = $t
     }
     $null = $stringBuilder.Append("</pre>")
     $stringBuilder.ToString()
 }    
 
 function Copy-ColoredHTML 
 {
     <#
     .Synopsis
         Copies the currently selected text in the current file as colorized HTML
     .Description
         Copies the currently selected text in the current file as colorized HTML
         This allows for a user to paste colorized scripts into web pages or blogging 
         software
     .Example
         Copy-ColoredHTML
     #>
     param()
     
 	$currentText = Select-CurrentText -NotInCommandPane -NotInOutput
 	if (-not $currentText) {
 		$currentFile = Get-CurrentOpenedFileText		
 		$text = $currentFile
 	} else {
 		$text = $currentText
 	}
 	if (-not $text) {  return }
 	
 	$sb = [ScriptBlock]::Create($text)
 	$Error | Select-object -last 1 | ogv
 	
 	$colorizedHTML = Write-ColorizedHTML -Text "$sb"
 	[Windows.Clipboard]::SetText($colorizedHTML )
 	return
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
     
 }
 
 
 Copy-Colored
 Write-ColorizedHTML
`

