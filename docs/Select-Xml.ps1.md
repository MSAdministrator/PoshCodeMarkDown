---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 2762
Published Date: 
Archived Date: 2011-07-06t05
---

#  - 

## Description

force $duds to be an array so for small documents exceptions aren’t thrown.

## Comments



## Usage



## TODO



## function

`select-xml`

## Code

`#
 #
 
 
 
 
 $xlr8r = [type]::gettype("System.Management.Automation.TypeAccelerators")
 $xlinq = [Reflection.Assembly]::Load("System.Xml.Linq, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 $xlinq.GetTypes() | ? { $_.IsPublic -and !$_.IsSerializable -and $_.Name -ne "Extensions" -and !$xlr8r::Get[$_.Name] } | % {
   $xlr8r::Add( $_.Name, $_.FullName )
 }
 if(!$xlr8r::Get["Stack"]) {
    $xlr8r::Add( "Stack", "System.Collections.Generic.Stack``1, System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" )
 }
 if(!$xlr8r::Get["Dictionary"]) {
    $xlr8r::Add( "Dictionary", "System.Collections.Generic.Dictionary``2, mscorlib, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" )
 }
 if(!$xlr8r::Get["PSParser"]) {
    $xlr8r::Add( "PSParser", "System.Management.Automation.PSParser, System.Management.Automation, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35" )
 }
 
 
 
 filter Format-XML {
 #.Synopsis
 #.Description
 #.Parameter Xml
 #.Parameter Path
 #.Parameter Indent
 #.Example
 #.Example
 #.Example
 #.Example
 #
 Param(
    [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="Document")]
    [xml]$Xml
 ,
    [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName="File")]
    [Alias("PsPath")]
    [string]$Path
 ,
    [Parameter(Mandatory=$false)]
    $Indent=2
 )
    if($Path) { [xml]$xml = Get-Content $Path }
    
    $StringWriter = New-Object System.IO.StringWriter
    $XmlWriter = New-Object System.Xml.XmlTextWriter $StringWriter
    $xmlWriter.Formatting = "indented"
    $xmlWriter.Indentation = $Indent
    $xml.WriteContentTo($XmlWriter)
    $XmlWriter.Flush()
    $StringWriter.Flush()
    Write-Output $StringWriter.ToString()
 }
 Set-Alias fxml Format-Xml -EA 0
 
 function Select-Xml {
 #.Synopsis
 #.Description
 #
 #.Parameter Content
 #.Parameter Namespace
 #.Parameter Path
 #.Parameter Xml
 #.Parameter XPath
 #.Parameter RemoveNamespace
 [CmdletBinding(DefaultParameterSetName="Xml")]
 PARAM(
    [Parameter(Position=1,ParameterSetName="Path",Mandatory=$true,ValueFromPipelineByPropertyName=$true)]
    [ValidateNotNullOrEmpty()]
    [Alias("PSPath")]
    [String[]]$Path
 ,
    [Parameter(Position=1,ParameterSetName="Xml",Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [ValidateNotNullOrEmpty()]
    [Alias("Node")]
    [System.Xml.XmlNode[]]$Xml
 ,
    [Parameter(ParameterSetName="Content",Mandatory=$true,ValueFromPipeline=$true)]
    [ValidateNotNullOrEmpty()]
    [String[]]$Content
 ,
    [Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false)]
    [ValidateNotNullOrEmpty()]
    [Alias("Query")]
    [String[]]$XPath
 ,
    [Parameter(Mandatory=$false)]
    [ValidateNotNullOrEmpty()]
    [Hashtable]$Namespace
 ,
    [Switch]$RemoveNamespace
 )
 BEGIN {
    function Select-Node {
    PARAM([Xml.XmlNode]$Xml, [String[]]$XPath, $NamespaceManager)
    BEGIN {
       foreach($node in $xml) {
          if($NamespaceManager -is [Hashtable]) {
             $nsManager = new-object System.Xml.XmlNamespaceManager $node.NameTable
             foreach($ns in $Namespace.GetEnumerator()) {
                $nsManager.AddNamespace( $ns.Key, $ns.Value )
             }
          }
          foreach($path in $xpath) {
             $node.SelectNodes($path, $NamespaceManager)
    }  }  }  }
 
    [Text.StringBuilder]$XmlContent = [String]::Empty
 }
 
 PROCESS {
    $NSM = $Null; if($PSBoundParameters.ContainsKey("Namespace")) { $NSM = $Namespace }
 
    switch($PSCmdlet.ParameterSetName) {
       "Content" {
          $null = $XmlContent.AppendLine( $Content -Join "`n" )
       }
       "Path" {
          foreach($file in Get-ChildItem $Path) {
             [Xml]$Xml = Get-Content $file
             if($RemoveNamespace) {
                $Xml = Remove-XmlNamespace $Xml
             }
             Select-Node $Xml $XPath  $NSM
          }
       }
       "Xml" {
          foreach($node in $Xml) {
             if($RemoveNamespace) {
                $node = Remove-XmlNamespace $node
             }
             Select-Node $node $XPath $NSM
          }
       }
    }
 }
 END {
    if($PSCmdlet.ParameterSetName -eq "Content") {
       [Xml]$Xml = $XmlContent.ToString()
       if($RemoveNamespace) {
          $Xml = Remove-XmlNamespace $Xml
       }
       Select-Node $Xml $XPath  $NSM
    }
 }
 
 }
 Set-Alias slxml Select-Xml -EA 0
 
 function Convert-Node {
 #.Synopsis 
 param(
 [Parameter(Mandatory=$true,ValueFromPipeline=$true)]
 [System.Xml.XmlReader]$XmlReader,
 [Parameter(Position=1,Mandatory=$true,ValueFromPipeline=$false)]
 [System.Xml.Xsl.XslCompiledTransform]$StyleSheet
 ) 
 PROCESS {
    $output = New-Object IO.StringWriter
    $StyleSheet.Transform( $XmlReader, $null, $output )
    Write-Output $output.ToString()
 }
 }
    
 function Convert-Xml {
 #.Synopsis
 #.Description
 #.Parameter Content
 #.Parameter Namespace
 #.Parameter Path
 #.Parameter Xml
 #.Parameter Xsl
 [CmdletBinding(DefaultParameterSetName="Xml")]
 PARAM(
    [Parameter(Position=1,ParameterSetName="Path",Mandatory=$true,ValueFromPipelineByPropertyName=$true)]
    [ValidateNotNullOrEmpty()]
    [Alias("PSPath")]
    [String[]]$Path
 ,
    [Parameter(Position=1,ParameterSetName="Xml",Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [ValidateNotNullOrEmpty()]
    [Alias("Node")]
    [System.Xml.XmlNode[]]$Xml
 ,
    [Parameter(ParameterSetName="Content",Mandatory=$true,ValueFromPipeline=$true)]
    [ValidateNotNullOrEmpty()]
    [String[]]$Content
 ,
    [Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false)]
    [ValidateNotNullOrEmpty()]
    [Alias("StyleSheet")]
    [String[]]$Xslt
 )
 BEGIN { 
    $StyleSheet = New-Object System.Xml.Xsl.XslCompiledTransform
    if(Test-Path @($Xslt)[0] -EA 0) { 
       Write-Verbose "Loading Stylesheet from $(Resolve-Path @($Xslt)[0])"
       $StyleSheet.Load( (Resolve-Path @($Xslt)[0]) )
    } else {
       Write-Verbose "$Xslt"
       $StyleSheet.Load(([System.Xml.XmlReader]::Create((New-Object System.IO.StringReader ($Xslt -join "`n")))))
    }
    [Text.StringBuilder]$XmlContent = [String]::Empty 
 }
 PROCESS {
    switch($PSCmdlet.ParameterSetName) {
       "Content" {
          $null = $XmlContent.AppendLine( $Content -Join "`n" )
       }
       "Path" {
          foreach($file in Get-ChildItem $Path) {
             Convert-Node -Xml ([System.Xml.XmlReader]::Create((Resolve-Path $file))) $StyleSheet
          }
       }
       "Xml" {
          foreach($node in $Xml) {
             Convert-Node -Xml (New-Object Xml.XmlNodeReader $node) $StyleSheet
          }
       }
    }
 }
 END {
    if($PSCmdlet.ParameterSetName -eq "Content") {
       [Xml]$Xml = $XmlContent.ToString()
       Convert-Node -Xml $Xml $StyleSheet
    }
 }
 }
 Set-Alias cvxml Convert-Xml -EA 0
 
 function Remove-XmlNamespace {
 #.Synopsis
 #.Description
 #.Parameter Content
 #.Parameter Path
 #
 #.Parameter Xml
 [CmdletBinding(DefaultParameterSetName="Xml")]
 PARAM(
    [Parameter(Position=1,ParameterSetName="Path",Mandatory=$true,ValueFromPipelineByPropertyName=$true)]
    [ValidateNotNullOrEmpty()]
    [Alias("PSPath")]
    [String[]]$Path
 ,
    [Parameter(Position=1,ParameterSetName="Xml",Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [ValidateNotNullOrEmpty()]
    [Alias("Node")]
    [System.Xml.XmlNode[]]$Xml
 ,
    [Parameter(ParameterSetName="Content",Mandatory=$true,ValueFromPipeline=$true)]
    [ValidateNotNullOrEmpty()]
    [String[]]$Content
 ,
    [Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false)]
    [ValidateNotNullOrEmpty()]
    [Alias("StyleSheet")]
    [String[]]$Xslt
 )
 BEGIN { 
    $StyleSheet = New-Object System.Xml.Xsl.XslCompiledTransform
    $StyleSheet.Load(([System.Xml.XmlReader]::Create((New-Object System.IO.StringReader @"
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
    [Text.StringBuilder]$XmlContent = [String]::Empty 
 }
 PROCESS {
    switch($PSCmdlet.ParameterSetName) {
       "Content" {
          $null = $XmlContent.AppendLine( $Content -Join "`n" )
       }
       "Path" {
          foreach($file in Get-ChildItem $Path) {
             [Xml]$Xml = Get-Content $file
             Convert-Node -Xml $Xml $StyleSheet
          }
       }
       "Xml" {
          $Xml | Convert-Node $StyleSheet
       }
    }
 }
 END {
    if($PSCmdlet.ParameterSetName -eq "Content") {
       [Xml]$Xml = $XmlContent.ToString()
       Convert-Node -Xml $Xml $StyleSheet
    }
 }
 }
 Set-Alias rmns Remove-XmlNamespace -EA 0
 
 
 
 function New-XDocument {
 #.Synopsis
 #.Description
 #.Parameter root
 #.Parameter version
 #.Parameter encoding
 #.Parameter standalone
 #.Parameter args
 #
 #.Example
 #
 #
 #
 #
 #
 #
 #
 #.Example 
 #
 #
 #
 #
 #
 Param(
    [Parameter(Mandatory = $true, Position = 0)]
    [System.Xml.Linq.XName]$root
 ,
    [Parameter(Mandatory = $false)]
    [string]$Version = "1.0"
 ,
    [Parameter(Mandatory = $false)]
    [string]$Encoding = "UTF-8"
 ,
    [Parameter(Mandatory = $false)]
    [string]$Standalone = "yes"
 ,
    [Parameter(Position=99, Mandatory = $false, ValueFromRemainingArguments=$true)]
    [PSObject[]]$args
 )
 BEGIN {
    $script:NameSpaceHash = New-Object 'Dictionary[String,XNamespace]'
    if($root.NamespaceName) {
       $script:NameSpaceHash.Add("", $root.Namespace)
    }
 }
 PROCESS {
    New-Object XDocument (New-Object XDeclaration $Version, $Encoding, $standalone),(
       New-Object XElement $(
          $root
          while($args) {
             $attrib, $value, $args = $args
             if($attrib -is [ScriptBlock]) {
                $attrib = ConvertFrom-XmlDsl $attrib
                Write-Verbose "Reparsed DSL: $attrib"
                &$attrib
             } elseif ( $value -is [ScriptBlock] -and "-Content".StartsWith($attrib)) {
                $value = ConvertFrom-XmlDsl $value
                &$value
             } elseif ( $value -is [XNamespace]) {
                New-Object XAttribute ([XNamespace]::Xmlns + $attrib.TrimStart("-")), $value
                $script:NameSpaceHash.Add($attrib.TrimStart("-"), $value)
             } else {
                New-Object XAttribute $attrib.TrimStart("-"), $value
             }
          }
       ))
 }
 }
 
 Set-Alias xml New-XDocument -EA 0
 Set-Alias New-Xml New-XDocument -EA 0
 
 function New-XAttribute {
 #.Synopsys
 #.Description
 #.Parameter name
 #.Parameter value
 Param([Parameter(Mandatory=$true)]$name,[Parameter(Mandatory=$true)]$value)
    New-Object XAttribute $name, $value
 }
 Set-Alias xa New-XAttribute -EA 0
 Set-Alias New-XmlAttribute New-XAttribute -EA 0
 
 
 function New-XElement {
 #.Synopsys
 #.Description
 #.Parameter tag
 #.Parameter args
 Param(
    [Parameter(Mandatory = $true, Position = 0)]
    [System.Xml.Linq.XName]$tag
 ,
    [Parameter(Position=99, Mandatory = $false, ValueFromRemainingArguments=$true)]
    [PSObject[]]$args
 )
 PROCESS {
   New-Object XElement $(
      $tag
      while($args) {
         $attrib, $value, $args = $args
            &$attrib
            &$value
         } elseif ( $value -is [XNamespace]) {
            New-Object XAttribute ([XNamespace]::Xmlns + $attrib.TrimStart("-")), $value
            $script:NameSpaceHash.Add($attrib.TrimStart("-"), $value)
         } else {
            New-Object XAttribute $attrib.TrimStart("-"), $value
         }        
      }
    )
 }
 }
 Set-Alias xe New-XElement
 Set-Alias New-XmlElement New-XElement
 
 function ConvertFrom-XmlDsl {
 Param([ScriptBlock]$script)
    $parserrors = $null
    $global:tokens = [PSParser]::Tokenize( $script, [ref]$parserrors )
    $duds = @( $global:tokens | Where-Object { $_.Type -eq "Command" -and !$_.Content.Contains('-') -and ($(Get-Command $_.Content -Type Cmdlet,Function,ExternalScript -EA 0) -eq $Null) } )
    [Array]::Reverse( $duds )
    
    [string[]]$ScriptText = "$script" -split "`n"
 
    ForEach($token in $duds ) {
       if( $token.Content.Contains(":") ) {
          $key, $localname = $token.Content -split ":"
          $ScriptText[($token.StartLine - 1)] = $ScriptText[($token.StartLine - 1)].Remove( $token.StartColumn -1, $token.Length ).Insert( $token.StartColumn -1, "'" + $($script:NameSpaceHash[$key] + $localname) + "'" )
       } else {
          $ScriptText[($token.StartLine - 1)] = $ScriptText[($token.StartLine - 1)].Remove( $token.StartColumn -1, $token.Length ).Insert( $token.StartColumn -1, "'" + $($script:NameSpaceHash[''] + $token.Content) + "'" )
       }
       $ScriptText[($token.StartLine - 1)] = $ScriptText[($token.StartLine - 1)].Insert( $token.StartColumn -1, "xe " )
    }
    Write-Output ([ScriptBlock]::Create( ($ScriptText -join "`n") ))
 }
    
 Export-ModuleMember -alias * -function New-XDocument, New-XAttribute, New-XElement, Remove-XmlNamespace, Convert-Xml, Select-Xml, Format-Xml
`

