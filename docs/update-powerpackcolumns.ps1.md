---
Author: dmitry sotnikov
Publisher: 
Copyright: 
Email: 
Version: 1.5.1
Encoding: ascii
License: cc0
PoshCode ID: 504
Published Date: 2009-08-06t07
Archived Date: 2013-06-22t18
---

# update-powerpackcolumns - 

## Description

this script is designed to work around a powerpack export issue in powergui 1.5.1 (fixed in subsequent releases)

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###################################################################
 ###################################################################
 ###################################################################
 ###################################################################
 
 ###################################################################
 ###################################################################
 $powergui_cfg_path = "$($env:APPDATA)\Quest Software\PowerGUI\quest.powergui.xml"
 $powerpack_path = 'c:\scripts\aliases.powerpack'
 $updated_powerpack_path = 'C:\scripts\new.powerpack'
 
 ##################################################
 ##################################################
 
 $pg = [xml] (gc $powergui_cfg_path)
 
 $uiviews = $null
 foreach ($c in $pg.configuration.items.container) {
 	if ($c.Name -eq 'UI View') { $uiviews = $c; break; }
 }
 
 $columns = @{}
 
 if ( $uiviews -ne $null ) {
 	foreach ($c in $uiviews.items.container.items.container) {
 		"Type: $($c.Type)"
 		$properties = @()
 		if ( $c.items.container -ne $null ) {
 			foreach ($col in $c.items.container) {$col.Name; $properties += $col.Name }
 			if ($properties.Length -gt 0) { $columns[$c.Type] = $properties }
 		} elseif ( $c.items.item -ne $null ) {
 			foreach ($col in $c.items.item) {$col.Name; $properties += $col.Name }
 			if ($properties.Length -gt 0) { $columns[$c.Type] = $properties }
 		} 
 	}
 	" === Hash Table === "
 	$columns
 }
 
 ##################################################################
 ##################################################################
 
 $pp = [xml] (gc $powerpack_path)
 
 function AssignTypes {
 	param($segment)
 	if ( $segment -ne $null ) {
 		$segment.returntype
 		if ($segment.items.container.name -notlike "properties*") {
 			" --- No columns assigned"
 			if ( ($segment.returntype -ne $null) -and $columns.ContainsKey($segment.returntype) ) {
 				" --- Found columns by type"
 				$cont = $pp.CreateElement("container")
 				$cont.SetAttribute("id", [Guid]::NewGuid().ToString())
 				$cont.SetAttribute("name", "properties_a807f902-4b43-4b22-86d8-724abc4c3d4a")
 				$segment["items"].AppendChild($cont)
 				$subitems = $pp.CreateElement("items")
 				$cont.AppendChild($subitems)
 				foreach ($c in $columns[$segment.returntype]) {
 					$item = $pp.CreateElement("item")
 					$item.SetAttribute("id", [Guid]::NewGuid().ToString())
 					$item.SetAttribute("name", $c)
 					$subitems.AppendChild($item)
 				}
 			}
 		} else {
 			" --- Columns already assigned"
 		}
 	}
 }
 
 function IterateTree {
 	param($segment)
 	if (($segment.Type -like 'Script*') -or ($segment.Type -like 'CmdLet*')) {
 		AssignTypes $segment
 	}
 	if ($segment.items.container -ne $null) {
 		$segment.items.container | ForEach-Object { IterateTree $_ }
 	}
 }
 
 " === Tree === "
 IterateTree $pp.configuration.items.container[0]
 
 " === Links === "
 $pp.configuration.items.container[1].items.container | 
 		where { $_.id -eq '481eccc0-43f8-47b8-9660-f100dff38e14' } | ForEach-Object {
 			$_.items.container | ForEach-Object { AssignTypes $_ }	
 			$_.items.item | ForEach-Object { AssignTypes $_ }	
 		}
 
 $pp.Save($updated_powerpack_path)
`

