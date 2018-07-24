---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6450
Published Date: 2016-07-19t14
Archived Date: 2016-07-27t03
---

# out-pptx - 

## Description

output text ato a new powerpoint slide

## Comments



## Usage



## TODO



## function

`out-pptx`

## Code

`#
 #
 Add-Type -AssemblyName Office
 Add-Type -AssemblyName Microsoft.Office.Interop.PowerPoint
 
 function Out-Pptx {
    [CmdletBinding()]
    param(
       [Parameter(ValueFromPipeline=$true)]
       $Content,
 
       $Presentation = "~\SkyDrive\Presentations\Intro to PowerShell.pptx"
    )
    begin {
       try {
          $PowerPoint = [System.Runtime.InteropServices.Marshal]::GetActiveObject('PowerPoint.Application')
       } catch {
          $PowerPoint = New-Object -ComObject PowerPoint.Application
          $PowerPoint.visible = [Microsoft.Office.Core.MsoTriState]::MsoTrue
          $Presentation = $PowerPoint.Presentations.Open( $Presentation )
       }
 
       if(!$Presentation) {
          $Last = $PowerPoint.Presentations.Count
          $Presentation = $PowerPoint.Presentations.Item($Last)
       }
    }
 
    process {
       $Count = $Presentation.Slides.Count
       $Slide = $Presentation.Slides.AddSlide($Count+1, $Presentation.Slides.Item($Count).CustomLayout)
 
       $slide.Shapes.Title.TextFrame.TextRange.Text = "PS> " + $MyInvocation.Line
       $Slide.Shapes.Item(2).TextFrame.TextRange.Text = $Content
    }
 }
`

