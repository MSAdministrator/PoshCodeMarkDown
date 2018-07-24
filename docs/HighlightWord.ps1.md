---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5731
Published Date: 2015-02-07t08
Archived Date: 2015-02-15t05
---

# highlightword - 

## Description

search a word docx document and highlight each instance of a pattern.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 [CmdletBinding()]
 param(
     [Parameter(Mandatory)]
     $FilePath,
     [Parameter(Mandatory)]
     $Pattern,
     $Color = "Yellow"
 )
 
 Add-Type -AssemblyName "DocumentFormat.OpenXml, Version=2.5.5631.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"
 
 $Color = [DocumentFormat.OpenXml.Wordprocessing.HighlightColorValues]$Color
 
 
 $Doc = [DocumentFormat.OpenXml.Packaging.WordprocessingDocument]::Open( $FilePath, $true )
 $Body = $Doc.MainDocumentPart.Document.Body
 
 $preserveSpace = New-Object documentformat.openxml.openxmlattribute "xml", "space","http://www.w3.org/XML/1998/namespace","preserve"
 
 foreach($Node in $Body.Descendants()){
     if($Node -is [DocumentFormat.OpenXml.Wordprocessing.Run] -and $Node.Text -match $Pattern) {
         $Parent = $Node.Parent
         $Previous = $Node
 
         $Blocks = $Node.Text -split $Pattern
         for($t=0;$t -lt $Blocks.Count;$t++) {
             $run = $Node.CloneNode($true)
 
             foreach($text in $run.Descendants()) { 
                 if($text -is [DocumentFormat.OpenXml.Wordprocessing.Text]) {
                     $text.Text = $Blocks[$t] 
                     $text.SetAttribute( $preserveSpace )
                     $text.RemoveNamespaceDeclaration("xml")
                 }
             }
 
             $Parent.InsertAfter($run, $Previous)
             $Previous = $Run
 
             if(($t + 1) -lt $Blocks.Count) {
                 $Run = $Node.CloneNode($true)
                 foreach($text in $run.Descendants()) { 
                     if($text -is [DocumentFormat.OpenXml.Wordprocessing.Text]) {
                         $text.Text = $Pattern
                     }
                 }
 
                 $hilite = New-Object DocumentFormat.OpenXml.Wordprocessing.Highlight
                 $hilite.Val = $Color
                 $RunProps = New-Object DocumentFormat.OpenXml.Wordprocessing.RunProperties
                 $RunProps.Append($hilite)
                 $Run.InsertAt($RunProps, 0)
                 $Parent.InsertAfter($Run, $Previous)
                 $Previous = $Run
             }
         }
         $Parent.RemoveChild($Node)
         Write-Verbose $Parent.OuterXml
     }
 }
 
 $Doc.Close()
`

