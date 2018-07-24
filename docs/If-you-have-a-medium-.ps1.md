---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 831
Published Date: 
Archived Date: 2016-05-31t08
---

#  - 

## Description

if you have a medium to large size website, you can provide a sitemap so that search engines intelligently index your site’s content.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ####################################################################################################
 ##
 ##
 ####################################################################################################
 ##
 ####################################################################################################
 
 $workingdir = "c:\scripts"
 
 
 
 
 function CreateElement([string]$url)
 {
 	$w = $global:writer
 	$w.WriteStartElement("url")
 	$w.WriteStartElement("loc")
 	$w.WriteString($url)
 
 }
 
 
 $encoding = [System.Text.Encoding]::UTF8
 $writer = New-Object System.Xml.XmlTextWriter( $Path, $encoding )
 $writer.Formatting = [system.xml.formatting]::indented
 
 Write-Host "`r`n`r`nGenerating $Domain Sitemap... $path"
 
 $writer.WriteStartDocument()
 
 $writer.WriteStartElement( "urlset" )
 $writer.WriteAttributeString( "xmlns", "http`://www.sitemaps.org/schemas/sitemap/0.9" )
 
 foreach ($static in $statics)
 {
 	CreateElement "$Domain/$Static"
 }
 
 foreach ($Location in $Locations)
 {
 	$r = "$domain/product/$location"
 	CreateElement $r
 }
 
 
 
 $writer.close()
 
 
 Write-Host "`n`rTotal number of URLs : $Counter"
`

