---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1492
Published Date: 2011-11-26t21
Archived Date: 2011-11-12t07
---

# remove-xmlnamespace - 

## Description

removes namespace definitions and prefixes from xml documents

## Comments



## Usage



## TODO



## function

`remove-xmlnamespace`

## Code

`#
 #
 function Remove-XmlNamespace {
 #.Synopsis
 #.Description
 #.Parameter Content
 #.Parameter Path
 #
 #.Parameter Xml
 #
 [CmdletBinding(DefaultParameterSetName="Xml")]
 PARAM( 
    [Parameter(ParameterSetName="Content",Mandatory=$true)]
    [String[]]$Content
 ,
    [Parameter(Position=0,ParameterSetName="Path",Mandatory=$true,ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [String[]]$Path
 ,
    [Parameter(Position=0,ParameterSetName="Xml",Mandatory=$true,ValueFromPipeline=$true)]
    [Alias("IO","InputObject")]
    [System.Xml.XmlDocument[]]$Xml
 )
 BEGIN {
    $xslt = New-Object System.Xml.Xsl.XslCompiledTransform
    $xslt.Load(([System.Xml.XmlReader]::Create((New-Object System.IO.StringReader @"
 <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:output method="xml" indent="yes"/>
    <xsl:template match="/|comment()|processing-instruction()">
       <xsl:copy>
          <xsl:apply-templates/>
       </xsl:copy>
    </xsl:template>
 
    <xsl:template match="*">
       <xsl:element name="{local-name()}">
          <xsl:apply-templates select="@*|node()"/>
       </xsl:element>
    </xsl:template>
 
    <xsl:template match="@*">
       <xsl:attribute name="{local-name()}">
          <xsl:value-of select="."/>
       </xsl:attribute>
    </xsl:template>
 </xsl:stylesheet>
 "@))))
 }
 PROCESS {
    switch($PSCmdlet.ParameterSetName) {
       "Content" {
          [System.Xml.XmlDocument[]]$Xml = @( [Xml]($Content -Join "`n") )
       }
       "Path" {
          [System.Xml.XmlDocument[]]$Xml = @()
          foreach($file in Get-ChildItem $Path) {
             $Xml += [Xml](Get-Content $file)
          }
       }
       "Xml" {
       }
    }
    foreach($input in $Xml) {
       $Output = New-Object System.Xml.XmlDocument
       $writer =$output.CreateNavigator().AppendChild()
       $xslt.Transform( $input.CreateNavigator(), $null, $writer )
       Write-Output $output
    }
 }
 }
`

