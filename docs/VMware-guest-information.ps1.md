---
Author: bruce shreffler
Publisher: 
Copyright: 
Email: 
Version: 4.1
Encoding: ascii
License: cc0
PoshCode ID: 3129
Published Date: 2012-12-27t10
Archived Date: 2016-03-04t19
---

# vmware guest information - 

## Description

i needed to write a script to generate a vmware guest inventory so i needed to know what was available within the powercli interface and where to find it. so i wrote this script to dump everything it could find about a single vmware guest. it has proved very useful to me. i hope others will also find it useful.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #
 #
 #
 #
 
 Param($guest)
 
 ####################################################################
 #
 #
 #
 
 function ProcessObject ($xobj, $pref){
     $plen = $pref.length
 	$pad = $width - $plen
 	If ($pad -lt 0){$pad = 0}
 	$xom = $xobj | get-member -membertype property
     foreach ($ent in $xom){
 		$xnam = $ent.name
         $xnlen = $xnam.length
         $strs = $ent.Definition.split(" ")
 		
 		
 		#
 		#
 		#
 		#
 		#
 		#
 		
         if ($xnam -match "^(CDDrives|FloppyDrives|HardDisks|NetworkAdapters|UsbDevices)$" ){
 			"{0,-$plen}{1,-1}{2,-$pad}{3,-2}{4,-30}" -f $pref, ".", $xnam, ":", "== skipping legacy device object ==" #| Out-File -FilePath $reportfile -Append
 		}
 		elseif ($xnam -match "^(ConsoleNic|PhysicalNic|ScsiLun|VirtualNic|VirtualSwitch)$" ){
 			"{0,-$plen}{1,-1}{2,-$pad}{3,-2}{4,-30}" -f $pref, ".", $xnam, ":", "== skipping legacy Network Adapter object ==" #| Out-File -FilePath $reportfile -Append
 		}
 		#
 		elseif ($xnam -match "^(Description|Host|HostId|State|VirtualMachineId)$" ){
 			"{0,-$plen}{1,-1}{2,-$pad}{3,-2}{4,-30}" -f $pref, ".", $xnam, ":", "== skipping obsolete identifier ==" #| Out-File -FilePath $reportfile -Append
 		}
 		#
 		elseif ($xnam -match "^(VMHost)$" ){
 			"{0,-$plen}{1,-1}{2,-$pad}{3,-2}{4,-30}" -f $pref, ".", $xnam, ":", "== skipping object to avoid infinite loop ==" #| Out-File -FilePath $reportfile -Append
 		}
 		elseif ($strs[0] -match "^System.String\[\]" ){
 			if (!$xobj.$xnam.count) {
 				$res = $xobj.$xnam
 				"{0,-$plen}{1,-1}{2,-$pad}{3,-2}{4,-30}" -f $pref, ".", $xnam, ":", $xobj.$xnam #| Out-File -FilePath $reportfile -Append
 			}
 			else {
 				$count = $xobj.$xnam.count
 				for ($i=0; $i -lt $count; $i++){
 					$namlen = $xnam.length
 					$ipad = $pad - 2 - $namlen
 					If ($i -gt 9){$ipad--}
 					If ($ipad -lt 1){$ipad = 1}
 					"{0,-$plen}{1,-1}{2,-$namlen}{3,1}{4,1}{5,-$ipad}{6,-2}{7,-30}" -f $pref, ".", $xnam, "[", $i, "]", ":", $xobj.$xnam[$i] #| Out-File -FilePath $reportfile -Append
 				}
 			}
 		}
 		elseif ($strs[0] -match "^System.(String|Int|Nullable|Boolean|DateTime|Double)" ){
 			if ($xnam -eq "value__"){
 				$vpad = $pad + 3
 				"{0,-$plen}{1,$vpad}{2,-2}{3,-2}{4,-30}" -f $pref, ": ", $xobj.$xnam, "-", $xobj #| Out-File -FilePath $reportfile -Append
 			}
 			else {
 				"{0,-$plen}{1,-1}{2,-$pad}{3,-2}{4,-30}" -f $pref, ".", $xnam, ":", $xobj.$xnam #| Out-File -FilePath $reportfile -Append		
 			}
 		}
         elseif ($strs[0] -match "^VMware.Vim(.|Automation.Types|Automation.ViCore)"){
 			if ($xnam -match "^(Parent|ParentSnapshot|VM)$"){
 				$pad = $width-$plen
 				If ($pad -lt 0){$pad = 0}
 				"{0,-$plen}{1,-1}{2,-$pad}{3,-2}" -f $pref, ".", $xnam, ": == skipping $xnam to avoid infinite loop ==" #| Out-File -FilePath $reportfile -Append
 			}
 			else {
 	            $pad = $width-$plen
 				If ($pad -lt 0){$pad = 0}
 	            $yobj = $xobj.$xnam
 				$newp = $pref + "." + $xnam
 				if (!$yobj){
 					"{0,-$plen}{1,-1}{2,-$pad}{3,-2}" -f $pref, ".", $xnam, ": =undefined=" #| Out-File -FilePath $reportfile -Append
 				}
 				else{
 					if (!$yobj.count){
 						ProcessObject $yobj $newp
 					}
 					else {
 						$lc = $yobj.count
 						for ($z = 0; $z -lt $lc; $z++){
 							$zobj = $yobj[$z]
 							$zperf = $newp + "[" + $z + "]"
 							ProcessObject $zobj $zperf
 						}
 					}
 				}
 			}
 		}
 		elseif ($strs[0] -match "^System.(Collections)" ){
 			Foreach ($entry in $xobj.$xnam){
 				$newpref = $pref + "." + $xnam
 				$cpad = $pad - $xnam.length - 1
 				If ($cpad -lt 0){$cpad = 0}
 				"{0,-$plen}{1,-1}{2,-$Cpad}{3,-2}{4,-30}" -f $newpref, ".", $entry.Key, ":", $entry.value #$strs[0] #| Out-File -FilePath $reportfile -Append
 			}
 		}
 		else {
             "{0,-$plen}{1,-1}{2,-$pad}{3,-2}{4,-30}" -f $pref, ".", $xnam, ":", $ent.Definition #$strs[0] #| Out-File -FilePath $reportfile -Append
 		}
 	}	
 }
 
 function POLauncher ($xobj, $pref){
 	if (!$xobj.count){
 		ProcessObject $xobj $pref
 	}
 	else {
 		$lc = $xobj.count
 		for ($z = 0; $z -lt $lc; $z++){
 			$zobj = $xobj[$z]
 			$zperf = $pref + "[" + $z + "]"
 			ProcessObject $zobj $zperf
 		}
 	}
 }
 
 
 
 $reportfile = "vminfo-rpt.txt"
 $first = 20
 $width = 70
 
 $xvm = get-vm -name $guest
 $prefix = $xvm.name + " Get-VM"
 POLauncher $xvm $prefix
 
 
 
 $dvm = get-HardDisk -vm $xvm
 $prefix = $xvm.name + " Get-HardDisk"
 POLauncher $dvm $prefix
 
 $cdvm = get-CDDrive -vm $xvm
 $prefix = $xvm.name + " Get-CDDrive"
 POLauncher $cdvm $prefix
 
 $flpvm = get-FloppyDrive -vm $xvm
 $prefix = $xvm.name + " Get-FloppyDrive"
 POLauncher $flpvm $prefix
 
 $netvm = get-NetworkAdapter -vm $xvm
 $prefix = $xvm.name + " Get-NetworkAdapter"
 POLauncher $netvm $prefix
 
 $usbvm = get-UsbDevice -vm $xvm
 $prefix = $xvm.name + " Get-UsbDevice"
 POLauncher usbvm $prefix
 
 $xview = $xvm | get-view
 $prefix = $xvm.name + " Get-View"
 POLauncher $xview $prefix
 
 $snap = get-snapshot $xvm
 if ($snap){
 	$prefix = $xvm.name + " Get-Snapshot"
 	POLauncher $snap $prefix
 }
 
 $vmhost = $xvm.VMHost
 $prefix = $vmhost.name + " Get-VMHost"
 POLauncher $vmhost $prefix
 
 #$vmhostv = Get-View $vmhost
 #$prefix = $vmhostv.name + " Get-View"
 
 
 $nics = Get-VMHostNetworkAdapter $vmhost
 foreach ($nic in $nics){
 	$prefix = $vmhost.name + " " + $nic.Name
 	POLauncher $nic $prefix
 }
 
 
 
 $vmguest = get-vmguest $xvm
 $prefix = $xvm.name + " Get-VMGuest"
 
 
 $vmdatastore = get-datastore -vm $xvm
 $prefix = $xvm.name + " Get-Datastore"
 POLauncher $vmdatastore $prefix
 
 $dc = Get-Datacenter -vm $xvm
 $prefix = $xvm.name + " Get-Datacenter"
`

