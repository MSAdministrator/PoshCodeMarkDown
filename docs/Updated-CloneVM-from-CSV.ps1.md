---
Author: monahancj
Publisher: 
Copyright: 
Email: 
Version: 4.0
Encoding: ascii
License: cc0
PoshCode ID: 3701
Published Date: 2014-10-19t09
Archived Date: 2014-06-22t09
---

# updated clonevm from csv - 

## Description

takes a fixed format csv and pumps out clones from a template.  easily modified to do clones of a vm or create new vms.  some of the restrictions mentioned in the notes appear to be fixed with vmware esx v4.1.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Param ($servercsv)
 
 
 <#
 .SYNOPSIS
 Mass cloning of virtual machines
 .DESCRIPTION
 Mass cloning of virtual machines from a template using a CSV file as a source.
 .NOTES
  1- Assumes template only has a C: drive and that only a D: will be added from the CSV file.  Additional drives will have to be done manually.
  2- Input CSV file format is at the bottom of this file.
  3- When testing use this command to mass delete the VMs created with the script.
  	 import-csv inputfile.csv | % { Remove-VM -DeleteFromDisk -VM $_.Name -RunAsync -Confirm:$false }
  4- Folder is not set due to the likelihood of duplicate folder names in large environment.  There's no good, consistent, and easy way that I could figure out to handle that.
  5- Creating the additional disk fails with PowerCLI 4.0 U1.  Reference URLs on the next two lines:
  	http://blog.vmpros.nl/2009/08/21/vmware-number-of-virtual-devices-exceeds-the-maximum-for-a-given-controller/
 	http://communities.vmware.com/thread/251601
  6- Setting the guest network fails with PowerCLI 4.0 U1.  I've posted the issue.
     http://communities.vmware.com/thread/262855
 	** UPDATE- Issue with dvSwitches not supported in PowerCli yet.  Workaround here- http://www.lucd.info/2010/03/04/dvswitch-scripting-part-8-get-and-set-network-adapters/
 	
 
 Todos-
   1- Set the guest network config. with either Copy-VMGuestfile and Invoke-VMScript, or Set-VMGuestNetworkInterface.
   2- Check/update vmware tools.  (May not be advisable, my conflict with sysprep reboot(s).)
   3- Create script to validate CSV file.  Possibly to be run by Project Managers.
   4- Create Excel spreadsheet with dropdown lists of cluster locations and other info to assist Project Managers.
 
 .LINK
 http://netops.ma.monster.com/virtech/VMware%20Wiki%20Library/Home.aspx
 .EXAMPLE
 C:\Temp> C:\ops\posh\VMware\Clone_Template_fromCSV.ps1 inputfile.csv
 #>
 
 Write-Output "`n++ $(Get-Date) - Starting Script."
 
 $ScriptDir = "\\vmscripthost201\repo"
 $VMTargets = Import-CSV $servercsv
 
 Write-Output "`n++ $(Get-Date) - Verifying server names are unique."
 
 $DupeNames = $false
 $AllVMs = Get-VM | Sort
 $i = 0;
 	$j = 0
    	while ($AllVMs[$j]) {
 		If ($VMTargets.Name -eq $AllVMs[$j].Name) {
 			$DupeNames = $true
 			$DupeNames
 			Write-Host "Requested VM name" $VMTargets.Name "is in use. `n"
 	$j++
 }
 	while ($VMTargets[$i]) {
 		$j = 0
    		while ($AllVMs[$j]) {
 			If ($VMTargets[$i].Name -eq $AllVMs[$j].Name) {
 				$DupeNames = $true
 				$DupeNames
 				Write-Host "Requested VM name" $VMTargets[$i].Name "is in use. `n"
 		$j++
 	$i++
 }
 
 
 If ($dupenames -eq $false) {
 	Write-Output "`n++ $(Get-Date) - Starting deploying."
 	$serverlist = Import-CSV $servercsv
 	$serverlist | % { 
 	$_.VC
     Connect-VIServer $_.VC | Select-Object Name
 	
 	If ($_.VC -match "ukvcenter101") {
 		If ($_.Template -match "std") {
 			$GuestIDs_mapto_OSCustSpec = Import-Csv $ScriptDir\StaticInfo\guestids_list_UK_Win2K8Std.csv }
 		ElseIf ($_.Template -match "ent") {
 			$GuestIDs_mapto_OSCustSpec = Import-Csv $ScriptDir\StaticInfo\guestids_list_UK_Win2K8Ent.csv }
 		Else { Write-Host "No matching UK template name to pick OSCustSpec"; break; }
 	}
 	Else { $GuestIDs_mapto_OSCustSpec = Import-Csv $ScriptDir\StaticInfo\guestids_list.csv }
 
 	$TemplateGuestId = (((Get-Template $_.Template).ExtensionData).Guest).GuestId
 	Write-Output "$($_.Template) and $($TemplateGuestId)"
 	$OSCustSpec = ($GuestIDs_mapto_OSCustSpec | ? { $_.guestid -eq $TemplateGuestId }).OSCustSpec
 	Write-Output "$($OSCustSpec)"
 	
 	$respool = $_.ResourcePool
 	$RPTarget = (Get-ResourcePool -location $_.Cluster | Where { $_.Name -match $respool } | get-random)
 	$VMHostTarget = (Get-VMHost -Location $_.Cluster | Where-Object { $_.ConnectionState -eq "Connected" } | Select Name,@{n="Lightest";e={ ( ($_.CpuUsageMhz/$_.CpuTotalMhz) + ($_.MemoryUsageMB/$_.MemoryTotalMB) )/2 }} | sort Lightest | select -first 1).Name
 	$DSTTarget = (.$ScriptDir\Get-DatastoreMostFree.ps1 $_.Cluster).Name
 	If ($DSTTarget -eq $null) { 
 		Write-Output "No datatstore returned for cluster $($_.Cluster).  Exiting script."
 		break
 	}
 	
 	Write-Output "`n++ $(Get-Date) `t- Creating VM:`t $(($_.Name).ToLower())`t$($_.Cluster)`t$($VMHostTarget)`t$($DSTTarget)`t$($RPTarget)`t$($OSCustSpec)"
 
 	New-VM -Confirm:$False `
            -Name ($_.Name).ToLower() `
            -Template $_.Template `
 		   -OSCustomizationSpec $OSCustSpec `
 		   -DiskStorageFormat Thin `
            -ResourcePool $RPTarget `
            -Description $_.Notes `
            -VMHost $VMHostTarget `
            -Datastore $DSTTarget
 	
 		
 	Set-VM -Confirm:$false `
            -VM $_.Name `
            -NumCpu $_.NumCpu `
            -MemoryMB ([int]$_.RAMSizeGB * 1024)
 	
 	If ( $_.Template -match "w2k8" ) { $SCSIControllerType = "VirtualLsiLogicSAS" } else { $SCSIControllerType = "VirtualLsiLogic" }
 	(Get-VM -Name $_.Name).Guest.OSFullName
 	$_.Template
 	$SCSIControllerType
 	New-HardDisk -VM $_.Name -CapacityKB ([int]$_.D_DriveSizeGB * 1024 * 1024) -DiskType Flat -StorageFormat Thin -Confirm:$false | New-ScsiController -Type $SCSIControllerType
 	
 	Get-NetworkAdapter -VM $_.Name | Set-NetworkAdapter -NetworkName $_.VLAN -Confirm:$false
 
 	Set-Annotation -Entity $_.Name -CustomAttribute Application -value $_.Application
 	Set-Annotation -Entity $_.Name -CustomAttribute "Business Unit" -value $_.BusinessUnit
 	Set-Annotation -Entity $_.Name -CustomAttribute Category -value $_.Category
 	Set-Annotation -Entity $_.Name -CustomAttribute Support -value $_.Support
 
 	Get-FloppyDrive -VM $_.Name | Remove-FloppyDrive -Confirm:$false
 	Get-UsbDevice -VM $_.Name | Remove-UsbDevice -Confirm:$false
 	
 	Get-VM -Name $_.Name | Get-VMResourceConfiguration | Set-VMResourceConfiguration -CPUReservationMhz 0 -CPULimitMhz $null
 	Get-VM -Name $_.Name | Get-VMResourceConfiguration | Set-VMResourceConfiguration -MemReservationMB  0 -MemLimitMB  $null
 
 	If (Get-Folder $_.Description) { Move-VM -VM $_.Name -Destination $_.Description }
 	ElseIf (Get-Folder Staging) { Move-VM -VM $_.Name -Destination Staging }
 	Else { "`n++ $(Get-Date) - Folder ""Staging"" does not exist.  The VM will be in the Datacenter root folder." }
   
   
 	
 	Write-Output "`n++ $(Get-Date) - Finished deploying."
 	
 	$serverlist | % { Get-VM $_.Name } | Select-Object @{n="Cluster";e={$_.VMHost.Parent.Name}},Name,ResourcePool,Folder,NumCpu,MemoryMB,@{n="ProvisionedSpaceGB";e={[int]$_.ProvisionedSpaceGB}},@{n="UsedSpaceGB";e={[int]$_.UsedSpaceGB}}
 
 
 Connect-VIServer $OrginalVIServer.Name
 
 <#
 VC,Cluster,Name,Template,ResourcePool,NumCPU,RAMSizeGB,D_DriveSizeGB,VLAN,Application,BusinessUnit,Category,Support,Notes
 vcenterxxx.fqdn.com,Bed-QADevIntv35,QA-DBHYPESS201,w2k8_std_x64_r2_tmpl,Normal,1,3,50,VLAN350,ESS,Hyperion,QA,Finance,"DEV00nnnnnn- For testing blah blah"
 #>
 
 Write-Output "`n++ $(Get-Date) - Finished Script."
`

