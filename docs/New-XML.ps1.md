---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 5348
Published Date: 2017-08-02t03
Archived Date: 2017-01-03t12
---

# new-xml - 

## Description

this is a first stab at creating a little dsl to generate xml. note that i used system.linq.xml (and output an xdocument) instead of the old xmldocument… this means you have to have .net 3.5 (linq) installed. it also means that if you want to be able to use the output via powershell’s magic xml dot-notation, you have to cast it to xmldocument, just write

## Comments



## Usage



## TODO



## function

`new-xml`

## Code

`#
 #
 
 $xlr8r = [type]::gettype("System.Management.Automation.TypeAccelerators")  
 $xlinq = [Reflection.Assembly]::Load("System.Xml.Linq, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")
 $xlinq.GetTypes() | ? { $_.IsPublic -and !$_.IsSerializable -and $_.Name -ne "Extensions" } | % {
    $xlr8r::Add( $_.Name, $_.FullName )
 }
 
 function New-Xml {
    Param([String]$root)
    New-Object XDocument (New-Object XDeclaration "1.0","utf-8","no"),(
       New-Object XElement $(
          $root
          while($args) {
             $attrib, $value, $args = $args
             if($attrib -is [ScriptBlock]) {
                &$attrib
             } elseif ( $value -is [ScriptBlock] -and "-Content".StartsWith($attrib)) {
                &$value
             } elseif ( $value -is [XNamespace]) {
                New-Object XAttribute ([XNamespace]::Xmlns + $attrib.TrimStart("-")),$value
             } else {
                New-Object XAttribute $attrib.TrimStart("-"), $value
             }
          }
       ))
 }
 
 function New-Element {
    Param([String]$tag)
    New-Object XElement $(
       $tag
       while($args) {
          $attrib, $value, $args = $args
          if($attrib -is [ScriptBlock]) {
             &$attrib
          } elseif ( $value -is [ScriptBlock] -and "-Content".StartsWith($attrib)) {
             &$value
          } elseif ( $value -is [XNamespace]) {
                New-Object XAttribute ([XNamespace]::Xmlns + $attrib.TrimStart("-")),$value
          } else {
             New-Object XAttribute $attrib.TrimStart("-"), $value
          }
       }
    )
 }
 Set-Alias xe New-Element
 
 
 ####################################################################################################
 #
 #
 #
`

