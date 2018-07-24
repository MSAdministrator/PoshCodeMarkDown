---
Author: kurt falde
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5876
Published Date: 2015-05-28t04
Archived Date: 2015-06-01t02
---

# evt xpath creation - 

## Description

parses a windows event log entry by specifying eventrecordid and log name and outputs xpath that can be used in an event filter/view or windows event forwarding filter

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 Script written to parse Event Log Entries to make usable Windows Event log filtering xpath for Windows Event Filters and Windows Eventlog Forwarding
 Finds all Nodes and Attributes that are not empty and not null and then recurses 3 levels up to find the 'Event' node and writes out the correct xpath
 Written 5/22/2015 - Kurt Falde
 #>
 
 
 $EventRecordIDToParse = "3139115"
 $xpath =  "*[System[EventRecordID=($EventRecordIDtoparse)]]"
 $EventToParse = Get-WinEvent -LogName 'Security' -FilterXPath "$xpath"
 [xml]$EventToParsexml = $EventToParse.ToXml()
 $nodes = $EventToParsexml | Select-Xml -XPath './/*'
 Foreach ($node in $nodes){
 
     
     $Nname = $node.Node.Name
     if($node.node.Parentnode.ParentNode.Name -eq "Event"){
     write-host "*[$($node.node.Parentnode.name)[($Nname=$Ntext)]]"}
     if($node.node.Parentnode.ParentNode.ParentNode.Name -eq "Event"){
     write-host "*[$($node.node.ParentNode.Parentnode.name)[$($node.node.parentnode.name)[($Nname=$Ntext)]]]"}
     if($node.node.Parentnode.ParentNode.ParentNode.Parentnode.Name -eq "Event"){
     write-host "*[$($node.node.ParentNode.Parentnode.Parentnode.name)[$($node.node.ParentNode.Parentnode.name)[$($node.node.parentnode.name)[($Nname=$Ntext)]]]]"}
   }
 
 
     $Nname = $node.Node.Name
     if($node.node.Parentnode.ParentNode.Name -eq "Event"){
     write-host "*[$($node.node.Parentnode.name)[$($node.node.LocalName)[@Name='$Nname'] and ($($node.node.LocalName)='$Ntext')]]"}
     if($node.node.Parentnode.ParentNode.ParentNode.Name -eq "Event"){
     write-host "*[$($node.node.ParentNode.Parentnode.name)[$($node.node.parentnode.name)[($Nname=$Ntext)]]]"}
     if($node.node.Parentnode.ParentNode.ParentNode.Parentnode.Name -eq "Event"){
     write-host "*[$($node.node.ParentNode.Parentnode.Parentnode.name)[$($node.node.ParentNode.Parentnode.name)[$($node.node.parentnode.name)[($Nname=$Ntext)]]]]"}
     }
 
 
     $AttributeText = ""
     $Attributes = $node.node.Attributes
     Foreach($Attribute in $Attributes){
     $AttrName = $Attribute.Name
     $AttributeText += "@$AttrName='$AttrText' and "
     }
     $AttributeText = $AttributeText.TrimEnd(" and ")
     $Nname = $node.Node.Name
     if($node.node.Parentnode.ParentNode.Name -eq "Event"){
     write-host "*[$($node.node.Parentnode.name)[$($node.node.LocalName)[$AttributeText]]"}
     if($node.node.Parentnode.ParentNode.ParentNode.Name -eq "Event"){
     write-host "*[$($node.node.ParentNode.Parentnode.name)[$($node.node.parentnode.name)[$AttributeText]]]"}
     if($node.node.Parentnode.ParentNode.ParentNode.Parentnode.Name -eq "Event"){
     write-host "*[$($node.node.ParentNode.Parentnode.Parentnode.name)[$($node.node.ParentNode.Parentnode.name)[$($node.node.parentnode.name)[$AttributeText]]]]"}
     }
 
 }
`

