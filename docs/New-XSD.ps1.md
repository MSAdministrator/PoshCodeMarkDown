---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 841
Published Date: 
Archived Date: 2009-02-05t17
---

# new-xsd - 

## Description

generates an xsd file with sqlxml annotations for a powershell object. the xsd file can be used with sqlxml assembly or com-based to automatically create a sql table and import the xml. use in conjunction with new-xml. see article at http

## Comments



## Usage



## TODO



## script

`get-sqltype`

## Code

`#
 #
 param($Object,$ItemTag="ITEM", $ChildItems="*", $Attributes=$Null)
 
 $header =  @"
 <?xml version="1.0" ?>
 <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:sql="urn:schemas-microsoft-com:mapping-schema">
 <xs:element name="ROOT" sql:is-constant="1">
 <xs:complexType>
 <xs:sequence>
 "@
 
 $footer  = @"
 </xs:sequence>
 </xs:complexType>
 </xs:element>
 </xs:schema>
 "@
 
 #######################
 function Get-SqlType
 {
     param($definition)
 
     $type = ([regex]'System\.([^ ]*) ').matches($definition) | %{$_.Groups[1].Value}
 
     switch ($type)
     {
         "Int64" {"bigint"}
         "Byte[]" {"varbinary"}
         "Boolean" {"bit"}
         "Decimal" {"decimal"}
         "DateTime" {"datetime"}
         "Double" {"float"}
         "Guid" {"uniqueidentifier"}
         "Int32" {"int"}
         "Single" {"real"}
         "Int16" {"smallint"}
         "Byte" {"tinyint"}
         default {"varchar(255)"}
     }
     
 
 #######################
     $xsd = $header
     $xsd += "`n   <xs:element name=`"$ItemTag`" sql:relation=`"$ItemTag`">`n"
     $xsd += "    <xs:complexType>`n"
     $xsd += "     <xs:sequence>`n"
     $seen = @()
     foreach ($prop in $Object | Get-Member -Type *Property $childItems)
     {
         $Name = $prop.Name
         if (!($seen -contains $Name))
         {
             $seen += $Name
             $xsd += "    <xs:element name=`"$Name`" sql:field=`"$Name`" sql:datatype=`"$(Get-SqlType $prop.Definition)`" />`n"
         }
     }
     $xsd += "    </xs:sequence>`n"
  
     if ($Attributes)
     {
         foreach ($attr in $Object | Get-Member -type *Property $attributes)
         {
             $Name = $attr.Name
             if (!($seen -contains $Name))
             {
                 $seen += $Name
                 $xsd += "    <xs:attribute name=`"$Name`" sql:field=`"$Name`" sql:datatype=`"$(Get-SqlType $attr.Definition)`" />`n"
             }
         }
     }
 
     $xsd += "    </xs:complexType>`n"
     $xsd += "   </xs:element>`n"
     $xsd += $footer
     $xsd
`

