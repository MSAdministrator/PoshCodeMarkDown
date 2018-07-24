---
Author: vercelj
Publisher: 
Copyright: 
Email: 
Version: 6.7
Encoding: ascii
License: cc0
PoshCode ID: 5927
Published Date: 2015-07-11t02
Archived Date: 2015-07-12t20
---

# xml module - 

## Description

forked from jaykul’s xml module 6.6

## Comments



## Usage



## TODO



## function

`add-accelerator`

## Code

`#
 #
 
 
 
 
 function Add-Accelerator {
 <#
    .Synopsis
       Add a type accelerator to the current session
    .Description
       The Add-Accelerator function allows you to add a simple type accelerator (like [regex]) for a longer type (like [System.Text.RegularExpressions.Regex]).
    .Example
       Add-Accelerator list System.Collections.Generic.List``1
       $list = New-Object list[string]
       
       Creates an accelerator for the generic List[T] collection type, and then creates a list of strings.
    .Example
       Add-Accelerator "List T", "GList" System.Collections.Generic.List``1
       $list = New-Object "list t[string]"
       
       Creates two accelerators for the Generic List[T] collection type.
    .Parameter Accelerator
       The short form accelerator should be just the name you want to use (without square brackets).
    .Parameter Type
       The type you want the accelerator to accelerate (without square brackets)
    .Notes
       When specifying multiple values for a parameter, use commas to separate the values. 
       For example, "-Accelerator string, regex".
       
       PowerShell requires arguments that are "types" to NOT have the square bracket type notation, because of the way the parsing engine works.  You can either just type in the type as System.Int64, or you can put parentheses around it to help the parser out: ([System.Int64])
 
       Also see the help for Get-Accelerator and Remove-Accelerator
    .Link
       http://huddledmasses.org/powershell-2-ctp3-custom-accelerators-finally/
 #>
 [CmdletBinding()]
 param(
    [Parameter(Position=0,ValueFromPipelineByPropertyName=$true)]
    [Alias("Key","Name")]
    [string[]]$Accelerator
 ,
    [Parameter(Position=1,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [Alias("Value","FullName")]
    [type]$Type
 )
 process {
    foreach($a in $Accelerator) { 
       if($xlr8r::AddReplace) { 
          $xlr8r::AddReplace( $a, $Type) 
       } else {
          $null = $xlr8r::Remove( $a )
          $xlr8r::Add( $a, $Type)
       }
       trap [System.Management.Automation.MethodInvocationException] {
          if($xlr8r::get.keys -contains $a) {
             if($xlr8r::get[$a] -ne $Type) {
                Write-Error "Cannot add accelerator [$a] for [$($Type.FullName)]`n                  [$a] is already defined as [$($xlr8r::get[$a].FullName)]"
             }
             Continue;
          } 
          throw
       }
    }
 }
 }
 
 &{ 
 $local:xlr8r = [psobject].assembly.gettype("System.Management.Automation.TypeAccelerators")
 $local:xlinq = [Reflection.Assembly]::Load("System.Xml.Linq, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 $xlinq.GetTypes() | ? { $_.IsPublic -and !$_.IsSerializable -and $_.Name -ne "Extensions" -and !$xlr8r::Get[$_.Name] } | Add-Accelerator
 
 Add-Accelerator "Dictionary" "System.Collections.Generic.Dictionary``2, mscorlib, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"
 Add-Accelerator "PSParser" "System.Management.Automation.PSParser, System.Management.Automation, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35"
 }
 
 
 function Get-XmlContent {
 #.Synopsis
 param(
     [Parameter(Position=1,Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
     [ValidateNotNullOrEmpty()]
     [Alias("PSPath","Path")]
     [String[]]$Content
 ,
     [Alias("Rn","Rm")]
     [Switch]$RemoveNamespace
 )
 begin {
     [Text.StringBuilder]$XmlContent = [String]::Empty
     [bool]$Path = $true
 }
 process {
     if($Path -and ($Path = Test-Path @($Content)[0] -EA 0)) { 
         foreach($file in Resolve-Path $Content) {
             $xml = New-Object System.Xml.XmlDocument;
             if($file.Provider.Name -eq "FileSystem") {
                 Write-Verbose $file.ProviderPath
                 $xml.Load( $file.ProviderPath )
             } else {
                 $ofs = "`n"
                 $xml.LoadXml( ([String](Get-Content $file)) )
             }
         }
         if($RemoveNamespace) {
             $xml.LoadXml( (Remove-XmlNamespace -Xml $xml.DocumentElement) )
         }
         Write-Output $xml
 
     } else {
         foreach($line in $content) {
             $null = $XmlContent.AppendLine( $line )
         }
     }
 }
 end {
     if(!$Path) {
         $xml = New-Object System.Xml.XmlDocument; 
         $xml.LoadXml( $XmlContent.ToString() )
         if($RemoveNamespace) {
             $xml.LoadXml( (Remove-XmlNamespace -Xml $xml.DocumentElement) )
         }
         Write-Output $xml
     }
 
 }}
 
 
 Set-Alias Import-Xml Get-XmlContent
 Set-Alias ipxml Get-XmlContent
 Set-Alias ipx Get-XmlContent
 Set-Alias Get-Xml Get-XmlContent
 Set-Alias gxml Get-XmlContent
 Set-Alias gx Get-XmlContent
 
 function Set-XmlContent {
 param(
     [Parameter(Mandatory=$true, Position=1)]
     [Alias("PSPath")]
     [String]$Path
 ,
     [Parameter(Position=5,ParameterSetName="Xml",Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
     [ValidateNotNullOrEmpty()]
     [Alias("Node")]
     [Xml]$Xml
 )
 process {
     $xml.Save( $Path )
 }
 }
 
 Set-Alias Export-Xml Set-XmlContent
 Set-Alias epxml Set-XmlContent
 Set-Alias epx Set-XmlContent
 Set-Alias Set-Xml Set-XmlContent
 Set-Alias sxml Set-XmlContent
 Set-Alias sx Set-XmlContent
 
 function Format-Xml {
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
 [CmdletBinding()]
 param(
    [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="Document")]
    [xml]$Xml
 ,
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName="File")]
    [Alias("PsPath")]
    [string]$Path
 ,
    [Parameter(Mandatory=$false)]
    $Indent=2
 )
 process {
    if($Path) { [xml]$xml = Get-Content $Path }
    
    $StringWriter = New-Object System.IO.StringWriter
    $XmlWriter = New-Object System.Xml.XmlTextWriter $StringWriter
    $xmlWriter.Formatting = "indented"
    $xmlWriter.Indentation = $Indent
    $xml.WriteContentTo($XmlWriter)
    $XmlWriter.Flush()
    $StringWriter.Flush()
    Write-Output $StringWriter.ToString()
 }}
 Set-Alias fxml Format-Xml -EA 0
 Set-Alias fx   Format-Xml -EA 0
 
 function Select-XmlNodeInternal {
 [CmdletBinding()]
 param([Xml.XmlNode[]]$Xml, [String[]]$XPath, [Hashtable]$NamespaceManager)
 begin {
     Write-Verbose "XPath = $($XPath -join ',')"
     foreach($node in $xml) {
         if($NamespaceManager) {
             $nsManager = new-object System.Xml.XmlNamespaceManager $node.NameTable
             foreach($ns in $NamespaceManager.GetEnumerator()) {
                 $nsManager.AddNamespace( $ns.Key, $ns.Value )
             }
             Write-Verbose "Names = $($nsManager | % { @{ $_ = $nsManager.LookupNamespace($_) } } | Out-String)"
         }
         foreach($path in $xpath) {
             $node.SelectNodes($path, $nsManager)
         }
     }
 }}
 
 function Select-Xml {
 #.Synopsis
 #.Description
 #
 [CmdletBinding(DefaultParameterSetName="Xml")]
 param(
     [Parameter(Position=1,Mandatory=$true,ValueFromPipeline=$false)]
     [ValidateNotNullOrEmpty()]
     [Alias("Query")]
     [String[]]$XPath
 ,
     [Parameter(ParameterSetName="Content",Mandatory=$true)]
     [ValidateNotNullOrEmpty()]
     [String[]]$Content
 ,
     [Parameter(Position=5,ParameterSetName="Path",Mandatory=$true,ValueFromPipelineByPropertyName=$true)]
     [ValidateNotNullOrEmpty()]
     [Alias("PSPath")]
     [String[]]$Path
 ,
     [Parameter(Position=5,ParameterSetName="Xml",Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
     [ValidateNotNullOrEmpty()]
     [Alias("Node")]
     [System.Xml.XmlNode[]]$Xml
 ,
     [Parameter(Position=10,Mandatory=$false)]
     [ValidateNotNullOrEmpty()]
     [Alias("Ns")]
     [Hashtable]$Namespace
 ,
     [Alias("Rn","Rm")]
     [Switch]$RemoveNamespace
 )
 begin {
     $NSM = $Null; if($PSBoundParameters.ContainsKey("Namespace")) { $NSM = $Namespace }
     $XmlNodes = New-Object System.Xml.XmlNode[] 1
     if($PSCmdlet.ParameterSetName -eq "Content") {
         $XmlNodes = Get-XmlContent $Content -RemoveNamespace:$RemoveNamespace
         Select-XmlNodeInternal $XmlNodes $XPath $NSM
     }
 }
 process {
     switch($PSCmdlet.ParameterSetName) {
         "Path" {
             $node = Get-XmlContent $Path -RemoveNamespace:$RemoveNamespace
             Select-XmlNodeInternal $node $XPath $NSM
         }
         "Xml" {
             foreach($node in $Xml) {
                 if($RemoveNamespace) {
                    [Xml]$node = Remove-XmlNamespace -Xml $node
                 }
                 Select-XmlNodeInternal $node $XPath $NSM
             }
         }
     }
 }}
 Set-Alias slxml Select-Xml -EA 0
 Set-Alias slx Select-Xml -EA 0
 
 
 function Update-Xml {
 #.Synopsis
 #.Description
 #
 #.Notes
 [CmdletBinding(DefaultParameterSetName="Set")]
 param(
     [Parameter(ParameterSetName="Before",Mandatory=$true)]
     [ValidateNotNullOrEmpty()]
     [Switch]$Before
 ,
     [Parameter(ParameterSetName="After",Mandatory=$true)]
     [ValidateNotNullOrEmpty()]
     [Switch]$After
 ,
     [Parameter(ParameterSetName="Append",Mandatory=$true)]
     [Switch]$Append
 ,
     [Parameter(ParameterSetName="Remove",Mandatory=$true)]
     [Switch]$Remove
 ,
     [Parameter(ParameterSetName="Replace",Mandatory=$true)]
     [Switch]$Replace
 ,
     [Parameter(Position=1,Mandatory=$true)]
     [ValidateNotNullOrEmpty()]
     [String[]]$XPath
 ,
     [Parameter(Position=2,Mandatory=$true,ValueFromPipeline=$false)]
     [ValidateNotNullOrEmpty()]
     [String]$Value
 ,
     [Parameter(Position=5,Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
     [ValidateNotNullOrEmpty()]
     [Alias("Node")]
     [System.Xml.XmlNode[]]$Xml
 ,   
     [Parameter(Position=10,Mandatory=$false)]
     [ValidateNotNullOrEmpty()]
     [Alias("Ns")]
     [Hashtable]$Namespace
 ,   
     [Switch]$Passthru
 )
 process
 {
     foreach($XmlNode in $Xml) {
         $select = @{}
         $select.Xml = $XmlNode
         $select.XPath = $XPath
         if($Namespace) {  
             $select.Namespace = $Namespace
         }
         $document =
             if($XmlNode -is [System.Xml.XmlDocument]) {
                 $XmlNode
             } else { 
                 $XmlNode.get_OwnerDocument()
             }
         if($xValue = $Value -as [Xml]) {
             $xValue = $document.ImportNode($xValue.SelectSingleNode("/*"), $true)
         }
         $nodes = Select-Xml @Select | Where-Object { $_ }
 
         if(@($nodes).Count -eq 0) { Write-Warning "No nodes matched your XPath, nothing will be updated" }
         
         foreach($node in $nodes) {
             $select.XPath = "$XPath/parent::*"
             $parent = Select-Xml @Select
             if(!$xValue) {
                 if($node -is [System.Xml.XmlAttribute] -and $Value.Contains("=")) {
                     $aName, $aValue = $Value.Split("=",2)
                     if($aName.Contains(":")){
                         $ns,$name = $aName.Split(":",2)
                         $xValue = $document.CreateAttribute( $name, $Namespace[$ns] )
                     } else {
                         $xValue = $document.CreateAttribute( $aName )
                     }
                     $xValue.Value = $aValue
                 }
             }
             
             switch($PSCmdlet.ParameterSetName) {
                 "Before" {
                     $null = $parent.InsertBefore( $xValue, $node )
                 }
                 "After" {
                     $null = $parent.InsertAfter( $xValue, $node )
                 }
                 "Append" {
                     $null = $parent.AppendChild( $xValue )
                 }
                 "Remove" {
                     $null = $parent.RemoveChild( $node )
                 }
                 "Replace" {
                     if(!$xValue) {
                         $xValue = $document.CreateTextNode( $Value )
                     }
                     $null = $parent.ReplaceChild( $xValue, $node )
                 }
                 "Set" {
                     } else {
                         if($node -is [System.Xml.XmlElement]) {
                             if(!$xValue) {
                                 $xValue = $document.CreateTextNode( $Value )
                             }
                             $null = $node.set_innerXml("")
                             $null = $node.AppendChild($xValue)
                         }
                         elseif($node -is [System.Xml.XmlAttribute]) {
                             $node.Value = $Value
                         } else {
                             Write-Warning "$XPath selects a node of type $($node.GetType()), which we haven't handled. Please add that handler!"
                         }
                     }
                 }
             }
         }
         if($Passthru) {
             Write-Output $XmlNode
         }
     }
 }}
 Set-Alias uxml Update-Xml -EA 0
 Set-Alias ux Update-Xml -EA 0
 
 function Convert-Node {
 #.Synopsis 
 [CmdletBinding(DefaultParameterSetName="Reader")]
 param(
    [Parameter(ParameterSetName="ByNode",Mandatory=$true,ValueFromPipeline=$true)]
    [System.Xml.XmlNode]$Node
 ,
    [Parameter(ParameterSetName="Reader",Mandatory=$true,ValueFromPipeline=$true)]
    [System.Xml.XmlReader]$XmlReader
 ,
    [Parameter(Position=1,Mandatory=$true,ValueFromPipeline=$false)]
    [System.Xml.Xsl.XslCompiledTransform]$StyleSheet
 ,
    [Parameter(Position=2,Mandatory=$false)]
    [Alias("Parameters")]
    [hashtable]$Arguments
 )
 PROCESS {
    if($PSCmdlet.ParameterSetName -eq "ByNode") {
       $XmlReader = New-Object Xml.XmlNodeReader $node
    }
 
    $output = New-Object IO.StringWriter
    $argList = $null
    
    if($Arguments) {
       $argList = New-Object System.Xml.Xsl.XsltArgumentList
       foreach($arg in $Arguments.GetEnumerator()) {
          $namespace, $name = $arg.Key -split ":"
          if(!$name) { 
             $name = $Namespace
             $namespace = ""
          }
          
          Write-Verbose "ns:$namespace name:$name value:$($arg.Value)"
          $argList.AddParam($name,"$namespace",$arg.Value)
       }
    }
    
    $StyleSheet.Transform( $XmlReader, $argList, $output )
    Write-Output $output.ToString()
 }
 }
 
 function Convert-Xml {
 #.Synopsis
 #.Description
 [CmdletBinding(DefaultParameterSetName="Xml")]
 param(
     [Parameter(Position=1,ParameterSetName="Xml",Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
     [ValidateNotNullOrEmpty()]
     [Alias("Node")]
     [System.Xml.XmlNode[]]$Xml
 ,   
     [Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false)]
     [ValidateNotNullOrEmpty()]
     [Alias("StyleSheet")]
     [String]$Xslt
 ,
     [Alias("Parameters")]
     [hashtable]$Arguments
 )
 begin { 
    $StyleSheet = New-Object System.Xml.Xsl.XslCompiledTransform
    if(Test-Path $Xslt -EA 0) { 
       Write-Verbose "Loading Stylesheet from $(Resolve-Path $Xslt)"
       $StyleSheet.Load( (Resolve-Path $Xslt) )
    } else {
       $OFS = "`n"
       Write-Verbose "$Xslt"
       $StyleSheet.Load(([System.Xml.XmlReader]::Create((New-Object System.IO.StringReader $Xslt))))
    }
 }
 process {
    foreach($node in $Xml) {
       Convert-Node -Xml (New-Object Xml.XmlNodeReader $node) $StyleSheet $Arguments
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
    [Parameter(Position=1,ParameterSetName="Xml",Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [ValidateNotNullOrEmpty()]
    [Alias("Node")]
    [System.Xml.XmlNode[]]$Xml
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
    $Xml | Convert-Node $StyleSheet
 }
 }
 Set-Alias rmns Remove-XmlNamespace -EA 0
 Set-Alias rmxns Remove-XmlNamespace -EA 0
 
 
 
 
 
 function Remove-XmlElement {
 #.Synopsis
 #.Description
 #.Parameter Content
 #.Parameter Path
 #
 #.Parameter Xml
 [CmdletBinding(DefaultParameterSetName="Xml")]
 PARAM(
    [Parameter(Position=0,ParameterSetName="Xml")] #,Mandatory=$true
    #[ValidateNotNullOrEmpty()]
    [XNamespace[]]$Namespace
 ,
    [Parameter(Position=1,ParameterSetName="Xml",Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [ValidateNotNullOrEmpty()]
    [Alias("Node")]
    [System.Xml.XmlNode[]]$Xml
 )
 BEGIN {
    foreach($Node in @($Xml)) {
       $Allspaces += Get-Namespace -Xml $Node
 
       $nsManager = new-object System.Xml.XmlNamespaceManager $node.NameTable
       foreach($ns in $Allspaces.GetEnumerator()) {
           $nsManager.AddNamespace( $ns.Key, $ns.Value )
       }
 
       if(!$Namespace) {
          $root = $Node.DocumentElement
          $nsManager.AddNamespace("compat", "http://schemas.openxmlformats.org/markup-compatibility/2006")
          if($ignorable = $root.SelectSingleNode("@compat:Ignorable",$nsManager)) {
             foreach($prefix in $ignorable.get_InnerText().Split(" ")) {
                $Namespace += $root.GetNamespaceOfPrefix($prefix)
             }
          }
       }
    }
 
    
    Write-Verbose "$Namespace"
    $i = 0
    $NSString = $(foreach($n in $Namespace) { "xmlns:n$i='$n'"; $i+=1 }) -Join " "
    $EmptyTransforms = $(for($i =0; $i -lt $Namespace.Count;$i++) {
       "<xsl:template match='n${i}:*'>
       </xsl:template>
       <xsl:template match='@n${i}:*'>
       </xsl:template>"
    })
    
    $XSLT = @"
 <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" $NSString>
    <xsl:output method="xml" indent="yes"/>
    <xsl:template match="@*|node()">
       <xsl:copy>
          <xsl:apply-templates select="@*|node()"/>
       </xsl:copy>
    </xsl:template>
    $EmptyTransforms
 </xsl:stylesheet>
 "@
    Write-Verbose $XSLT
  
    $StyleSheet = New-Object System.Xml.Xsl.XslCompiledTransform
    $StyleSheet.Load(([System.Xml.XmlReader]::Create((New-Object System.IO.StringReader $XSLT))))
    [Text.StringBuilder]$XmlContent = [String]::Empty 
 }
 PROCESS {
    $Xml | Convert-Node $StyleSheet
 }
 }
 
 function Get-Namespace {
 param(
    [Parameter(Position=0)]
    [String[]]$Prefix = "*"
 ,
    [Parameter(Position=1,ParameterSetName="Xml",Mandatory=$true,ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
    [ValidateNotNullOrEmpty()]
    [Alias("Node")]
    [System.Xml.XmlNode[]]$Xml
 )
    foreach($Node in @($Xml)) {
       $results = @{}
       if($Node -is [Xml.XmlDocument]) {
          $Node = $Node.DocumentElement
       }
       foreach($ns in $Node.CreateNavigator().GetNamespacesInScope("All").GetEnumerator()) {
          foreach($p in $Prefix) {
             if($ns.Key -like $p) {
                $results.Add($ns.Key, $ns.Value)
                break;
             }
          }
       }
       $results
    }
 }
 
 
 
 
 function ConvertFrom-CliXml {
    param(
       [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true)]
       [ValidateNotNullOrEmpty()]
       [String[]]$InputObject
    )
    begin
    {
       $OFS = "`n"
       [String]$xmlString = ""
    }
    process
    {
       $xmlString += $InputObject
    }
    end
    {
       $type = [psobject].assembly.gettype("System.Management.Automation.Deserializer")
       $ctor = $type.getconstructor("instance,nonpublic", $null, @([xml.xmlreader]), $null)
       $sr = new-object System.IO.StringReader $xmlString
       $xr = new-object System.Xml.XmlTextReader $sr
       $deserializer = $ctor.invoke($xr)
       $method = @($type.getmethods("nonpublic,instance") | where-object {$_.name -like "Deserialize"})[1]
       $done = $type.getmethod("Done", [System.Reflection.BindingFlags]"nonpublic,instance")
       while (!$done.invoke($deserializer, @()))
       {
          try {
             $method.invoke($deserializer, "")
          } catch {
             write-warning "Could not deserialize $xmlString"
          }
       }
       $xr.Close()
       $sr.Dispose()
    }
 }
 
 function ConvertTo-CliXml {
    param(
       [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true)]
       [ValidateNotNullOrEmpty()]
       [PSObject[]]$InputObject
    )
    begin {
       $type = [psobject].assembly.gettype("System.Management.Automation.Serializer")
       $ctor = $type.getconstructor("instance,nonpublic", $null, @([System.Xml.XmlWriter]), $null)
       $sw = new-object System.IO.StringWriter
       $xw = new-object System.Xml.XmlTextWriter $sw
       $serializer = $ctor.invoke($xw)
       $method = $type.getmethod("Serialize", "nonpublic,instance", $null, [type[]]@([object]), $null)
       $done = $type.getmethod("Done", [System.Reflection.BindingFlags]"nonpublic,instance")
    }
    process {
       try {
          [void]$method.invoke($serializer, $InputObject)
       } catch {
          write-warning "Could not serialize $($InputObject.gettype()): $InputObject"
       }
    }
    end {    
       [void]$done.invoke($serializer, @())
       $sw.ToString()
       $xw.Close()
       $sw.Dispose()
    }
 }
 
 
 
 function New-XDocument {
 #.Synopsis
 #.Description
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
 [CmdletBinding()]
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
    [Parameter(Mandatory = $false)]
    [hashtable]$Parameters
 ,
    [AllowNull()][AllowEmptyString()][AllowEmptyCollection()]
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
    if($Parameters) {
       foreach($key in $Parameters.Keys) {
          Set-Variable $key $Parameters.$key -Scope Script
       }
    }
    New-Object XDocument (New-Object XDeclaration $Version, $Encoding, $standalone),(
       New-Object XElement $(
          $root
          while($args) {
             $attrib, $value, $args = $args
             if($attrib -is [ScriptBlock]) {
                $attrib = ConvertFrom-XmlDsl $attrib
                Write-Verbose "Reparsed DSL: $attrib"
                & $attrib
             } elseif ( $value -is [ScriptBlock] -and "-DSL".StartsWith($attrib.TrimEnd(':').ToUpper())) {
                $value = ConvertFrom-XmlDsl $value
                Write-Verbose "Reparsed DSL: $value"
                & $value
             } elseif ( $value -is [ScriptBlock] -and "-CONTENT".StartsWith($attrib.TrimEnd(':').ToUpper())) {
                & $value
             } elseif ( $value -is [XNamespace]) {
                New-Object XAttribute ([XNamespace]::Xmlns + $attrib.TrimStart("-").TrimEnd(':')), $value
                $script:NameSpaceHash.Add($attrib.TrimStart("-").TrimEnd(':'), $value)
             } else {
                Write-Verbose "XAttribute $attrib = $value"
                New-Object XAttribute $attrib.TrimStart("-").TrimEnd(':'), $value
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
 [CmdletBinding()]
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
 [CmdletBinding()]
 Param(
    [Parameter(Mandatory = $true, Position = 0)]
    [System.Xml.Linq.XName]$tag
 ,
    [AllowNull()][AllowEmptyString()][AllowEmptyCollection()]
    [Parameter(Position=99, Mandatory = $false, ValueFromRemainingArguments=$true)]
    [PSObject[]]$args
 )
 PROCESS {
   New-Object XElement $(
      $tag
      Write-Verbose "New-XElement $tag $($args -join ',')"
      while($args) {
         $attrib, $value, $args = $args
            & $attrib
            & $value
         } elseif ( $value -is [XNamespace]) {
            Write-Verbose "New XAttribute xmlns: $($attrib.TrimStart("-").TrimEnd(':')) = $value"
            New-Object XAttribute ([XNamespace]::Xmlns + $attrib.TrimStart("-").TrimEnd(':')), $value
            $script:NameSpaceHash.Add($attrib.TrimStart("-").TrimEnd(':'), $value)
         } elseif($value -match "^-(?!\d)\w") {
             $args = @($value)+@($args)
         } elseif($value -ne $null) {
            Write-Verbose "New XAttribute $($attrib.TrimStart("-").TrimEnd(':')) = $value"
            New-Object XAttribute $attrib.TrimStart("-").TrimEnd(':'), $value
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
    [Array]$duds = $global:tokens | Where-Object { $_.Type -eq "Command" -and !$_.Content.Contains('-') -and ($(Get-Command $_.Content -Type Alias,Cmdlet,Function,ExternalScript -EA 0) -eq $Null) }
    if($duds) {
      [Array]::Reverse( $duds )
    }
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
 
 
 
     
    
 Export-ModuleMember -alias * -function New-XDocument, New-XAttribute, New-XElement, Remove-XmlNamespace, Remove-XmlElement, Get-Namespace, Get-XmlContent, Set-XmlContent, Convert-Xml, Select-Xml, Update-Xml, Format-Xml, ConvertTo-CliXml, ConvertFrom-CliXml
 
`

