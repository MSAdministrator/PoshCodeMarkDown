---
Author: don jones
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5256
Published Date: 2014-06-20t17
Archived Date: 2014-06-26t09
---

# corptools view - 

## Description

corp.format.ps1xml

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <?xml version="1.0" encoding="utf-8" ?>
 <Configuration>
     <ViewDefinitions>
 
         <View>
             <Name>Corp.NetInfo</Name>
             <ViewSelectedBy>
                 <TypeName>Corp.NetInfo</TypeName>
             </ViewSelectedBy>
  
              <TableControl>
 
                 <TableHeaders>
                     <TableColumnHeader>
                         <Label>ComputerName</Label>
                         <Width>16</Width>
                     </TableColumnHeader>
                     <TableColumnHeader>
                         <Label>Adapter</Label>
                         <Width>16</Width>
                     </TableColumnHeader>
                     <TableColumnHeader>
                         <Label>IPAddress</Label>
                         <Width>20</Width>
                     </TableColumnHeader>
                 </TableHeaders>
 
                 <TableRowEntries>
                     <TableRowEntry>
                         <TableColumnItems>
                             <TableColumnItem>
                                 <PropertyName>ComputerName</PropertyName>
                             </TableColumnItem>
                             <TableColumnItem>
                                 <PropertyName>Adapter</PropertyName>
                             </TableColumnItem>
                             <TableColumnItem>
                                 <ScriptBlock>"$($_.IPAddress)/$($_.Subnet)"</ScriptBlock>
                             </TableColumnItem>
                         </TableColumnItems>
                     </TableRowEntry>
                  </TableRowEntries>
                 
             </TableControl>
  
         </View>
 
     </ViewDefinitions>
 </Configuration>
`

