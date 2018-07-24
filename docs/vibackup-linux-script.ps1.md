---
Author: patrick
Publisher: 
Copyright: 
Email: 
Version: 0.4
Encoding: ascii
License: cc0
PoshCode ID: 938
Published Date: 2009-03-12t09
Archived Date: 2016-05-20t16
---

# vibackup linux script - 

## Description

generate vm ware backups via this script.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Param (
 	$viServer,
 	$bakVM,
 	$lxDest,
 )
 
 if (!$viServer) { $viServer = Read-Host -Prompt "VI Server " }
 if (!$bakVM) { $bakVM = Read-Host -Prompt "VM to Backup " }
 if (!$lxDest) { $lxDest = Read-Host -Prompt "Backup Path (ex. /srv/backup) " }
 
 $encoding = "OEM"
 $version = "0.4"
 $scriptout = @()
 
 $viUser = "vmware"
 $viPass = "vmware"
 
 Write-Host "viBackup Script Generator - " $version
 Write-Host "--------------------------------------------"
 
 if (($vmCon = Connect-VIServer -Protocol https $viServer) -eq "") { exit }
 $vm = Get-VM $bakVM -Server $vmCon -ErrorAction SilentlyContinue
 
 if (!$vm) {
 	Write-Host "No VM found."
 	Write-Host "Enter the Display Name from the VI Center:" -NoNewline
 	$tvm = Read-Host
 	if (!($vm=Get-VM $tvm -ErrorAction SilentlyContinue)) {
 		return $false
 		exit
 	}
 	Write-Host "You have entered 2 different Names. Please check that the VMX and the VM Name are the same!"
 	Write-Host -ForegroundColor yellow "Entered VMX Name: {$bakVM}"
 	Write-Host -ForegroundColor yellow "Entered VM Name: {$tvm}"
 	Write-Host -ForegroundColor yellow "Returned VM Name: {$vm}"
 	Write-Host -ForegroundColor yellow "IS THIS CORRECT (Yes/No)?:" -NoNewline
 	$sure = Read-Host
 	if ($sure -ne "yes") { exit }
 }
 
 if ((Get-Snapshot -VM $vm -Server $vmCon | Measure-Object | select count).Count -ne "0") {
 	Write-Host "VM has Snapshots. No Backup possible."
 	return $false
 	exit
 }
 
 $scriptname = $vm.name + ".sh"
 $vmname = $vm.Name
 $vmhost = $vm.Host
 
 
 $header = "#!/bin/bash"
 $user = "USER=`"${viUser}`""
 $pass = "PASS=`"${viPass}`""
 $dest = "DEST=`"${lxDest}`""
 $lxVM = "VM=`"${vmname}`""
 
 
 $scriptout += $header 
 $scriptout += $user 
 $scriptout += $pass 
 $scriptout += $dest 
 $scriptout += $lxvm 
 $scriptout += "" 
 $scriptout += "BKUP=``vmrun -h https://${viserver}/sdk -u `${USER} -p `${PASS} list | grep `${VM}`` " 
 $scriptout += "SNAPCHECK=``vmware-cmd -H ${viserver} -T ${vmhost} -U `${USER} -P `${PASS} `"`${BKUP}`" hassnapshot`` "
 $scriptout += "if [ `"`$SNAPCHECK`" != `"hassnapshot () =`" ]; then `n echo 'VM has a Snapshot. Arborting...' `n exit `n fi" 
 $scriptout += "vmrun -T esx -h https://${viserver}/sdk -u `${USER} -p `${PASS} snapshot `"`${BKUP}`" vmBackup"
 
 foreach ($hd in $vm.Harddisks) {
 	$hdname = $hd.Filename
 	$vifs = "vifs --server ${viserver} --dc 'KM' --username `${USER} --password `${PASS}  --get `"``echo $hdname | sed 's/.vmdk/-flat.vmdk/g'```" `${DEST}/`${VM}.vmdk "
 	$scriptout += $vifs
 }
 
 $scriptout += "vmrun -T esx -h https://${viserver}/sdk -u `${USER} -p `${PASS} deleteSnapshot `"`${BKUP}`" vmBackup" 
 
 $scriptout | Out-File $scriptname -Encoding $encoding
 
 Write-Host "Finished"
 Write-Host "Don't forget to convert the script under *nix/Linux with dos2unix!"
`

