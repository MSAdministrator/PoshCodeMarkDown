---
Author: lucd
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1312
Published Date: 
Archived Date: 2009-09-16t23
---

# vprofile - 

## Description

the script will export recursively all objects and properties of a vi/vsphere entity to a xml file.

## Comments



## Usage



## TODO



## script

`new-xmlnode`

## Code

`#
 #
 #
 
 $entityName = "Clus1"
 $childShow = $true
 $configShow = $false
 $vmDump = $true
 $profileDir = "C:\"
 
 $vInventory = [xml]"<Inventory></Inventory>"
 
 function New-XmlNode{
 	param($node, $nodeName)
 
 	$tmp = $global:vInventory.CreateElement($nodeName)
 	$node.AppendChild($tmp)
 }
 
 function Set-XmlAttribute{
 	param($node, $name, $value)
 
 	$node.SetAttribute($name, $value)
 }
 
 function Get-XmlNode{
 	param($path)
 
 	$global:vInventory.SelectNodes($path)
 }
 
 function Get-Properties{
 	param($BaseName, $Property, $XMLnode)
 
 	if(-not($Property -match "DynamicProperty$|DynamicType$")){
 
 		if($Property.Length -eq 0){
 			$XMLnewnode = New-XmlNode $XMLnode $BaseName.Substring(1)
 		}
 		else{
 			$XMLnewnode = New-XmlNode $XMLnode $Property
 		}
 
 		if((Invoke-Expression $BaseName) -ne $null){
 			$result = Invoke-Expression $BaseName
 			$type = Invoke-Expression ($BaseName + ".GetType()")
 			Set-XmlAttribute $XMLnewnode "Type" $type.Name
 			if($type.IsArray){
 				$ArrCount = Invoke-Expression ($BaseName + ".Count")
 				Set-XmlAttribute $XMLnewnode "Count" $ArrCount
 				for($i = 0; $i -lt $ArrCount; $i++){
 					Get-Properties ($BaseName + "[" + $i + "]") ($Property + "-" + $i + "-") $XMLnewnode
 				}
 			}
 			elseif($type.IsPrimitive -or $type.Name -eq "String" -or $type.Name -eq "DateTime"){
 				Set-XmlAttribute $XMLnewnode "Value" $result
 			}
 			else{
 				$props = $result | gm -memberType Property,NoteProperty
 				foreach($prop in $props){
 					Get-Properties ($BaseName + "." + $prop.Name) $prop.Name $XMLnewnode
 				}
 			}
 		}
 	}
 }
 
 function Get-Configuration{
 	param($object, $XMLnode, $label)
 
 	New-Variable -Name $label -Value $object
 	$properties = Get-Properties ('$' + $label) '' $XMLnode
 }
 
 function Get-Children{
 	param($entity, $path, $XMLnode, $recurse, $config)
 
 	if("vm","host" -notcontains $entity.Name){
 		$path += ("/" + $entity.Name)
 	}
 	switch -regex ($entity.gettype().name){
 		"Folder" {
 			if("vm","host" -notcontains $entity.Name){
 				$XMLnewnode = New-XmlNode $XMLnode "Folder" 
 				Set-XmlAttribute $XMLnewnode "Name" $entity.Name
 			}
 			if($recurse -and $entity.ChildEntity -ne $null){
 				foreach($childfld in (Get-View -Id $entity.ChildEntity)){
 					Get-Children $childfld $path $XMLnewnode $recurse $config
 				}
 			}
 		}
 		"VirtualMachine"{
 			if($vmDump){
 				if($entity.Config.Template){$VMtype = "Template"} else {$VMtype = "VM"}
 				$XMLnewnode = New-XmlNode $XMLnode $VMtype
 				Set-XmlAttribute $XMLnewnode "Name" $entity.Name
 				if($config){
 					Get-Configuration $entity.Config $XMLnewnode "Config"
 				}
 			}
 		}
 		"Datacenter"{
 			$XMLnewnode = New-XmlNode $XMLnode "Datacenter" 
 			Set-XmlAttribute $XMLnewnode "Name" $entity.Name
 			if($recurse -and $entity.HostFolder -ne $null){
 				foreach($childfld in (Get-View -Id $entity.HostFolder)){
 					Get-Children $childfld $path $XMLnewnode $recurse $config
 				}
 			}
 		}
 		"HostSystem|^ComputeResource$"{
 			$XMLnewnode = New-XmlNode $XMLnode "Host" 
 			Set-XmlAttribute $XMLnewnode "Name" $entity.Name
 			if($config){
 				Get-Configuration $entity.Config $XMLnewnode "Config"
 			}
 			if($recurse -and $entity.ChildEntity -ne $null){
 				foreach($childfld in (Get-View -Id $entity.ChildEntity)){
 					Get-Children $childfld $path $XMLnewnode $recurse $config
 				}
 			}
 		}
 		"ClusterComputeResource" {
 			$XMLnewnode = New-XmlNode $XMLnode "Cluster" 
 			Set-XmlAttribute $XMLnewnode "Name" $entity.Name
 			if($config){
 				Get-Configuration $entity.ConfigurationEx $XMLnewnode "ConfigurationEx"
 			}
 			if($recurse){
 				foreach($esx in (Get-View -Id $entity.Host)){
 					Get-Children $esx $path $XMLnewnode $recurse $config
 				}
 
 				$resourcePoolparent = Get-View -Id $entity.ResourcePool
 				if($resourcePoolParent.Vm -ne $null){
 					foreach($vm in (Get-View -Id $resourcePoolParent.Vm)){
 						Get-Children $vm $path $XMLnewnode $recurse $config
 					}
 				}
 				if($resourcePoolParent.resourcePool -ne $null){
 					foreach($respool in (Get-View -Id $resourcePoolParent.resourcePool)){
 						Get-Children $respool $path $XMLnewnode $recurse $config
 					}
 				}
 			}
 		}
 		"ResourcePool"{
 			$XMLnewnode = New-XmlNode $XMLnode "ResourcePool"
 			Set-XmlAttribute $XMLnewnode "Name" $entity.Name
 			if($config){
 				Get-Configuration $entity.Config $XMLnewnode "Config"
 			}
 			if($recurse -and $entity.Vm -ne $null){
 				foreach($childfld in (Get-View -Id $entity.Vm)){
 					Get-Children $childfld $path $XMLnewnode $recurse $config
 				}
 			}
 		}
 		Default{
 			write-host "Unhandled type" $entity.gettype().name
 		}
 	}
 }
 
 Get-Inventory -Name $entityName | Get-View | % {
 	Get-Children $_ $path (Get-XmlNode "Inventory") $childShow $configShow
 }
 
 $vInventory.Save($profileDir + "vProfile-" + $entityName + ".xml")
`

