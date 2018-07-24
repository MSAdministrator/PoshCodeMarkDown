---
Author: lucd
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1331
Published Date: 
Archived Date: 2009-09-28t13
---

# import-vcbimage - 

## Description

script that will use the converter to import a vcb created disk image into a datacenter

## Comments



## Usage



## TODO



## function

`invoke-cmd`

## Code

`#
 #
 #
 #
 #
 #
 #
 $I2VImageDir = <directory where the VCB images are stored>
 $I2VShare = <Sharename of the $I2VImageDir directory>
 $tgtDatacenter = <Target-datacenter>
 $I2Vuser = <account with access to the image directory and to the datacenter>
 $I2Vpassword = <password of the $I2Vuser account>
 $I2Vhost = <hostname where the images are stored>
 $ConvProgDir = "$env:ProgramFiles (x86)\VMware\Infrastructure\Converter Enterprise" 
 $ConvService = "vmware-converter" 
 $I2Vprog = "converter-tool.exe" 
 
 $p2v = [xml]@"
 <p2v version="2.2" xmlns="http://www.vmware.com/v2/sysimage/p2v" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.vmware.com/v2/sysimage/p2v p2vJob.xsd" xsi:type="P2VJob">
   <source>
     <hostedSpec networkPassword="" networkUsername="" path=""/>
   </source>
   <dest>
     <managedSpec datastore="" folder="" host="" resourcePool="" vmName="">
       <creds host="" port="0" type="sessionId" username="" password="" />
     </managedSpec>
   </dest>
   <importParams diskType="VMFS" preserveHWInfo="true" removeSystemRestore="false" targetProductVersion="PRODUCT_MANAGED">
     <nicMappings/>
     <diskLocations/>
   </importParams>
   <postProcessingParams/>
 </p2v>
 "@
 
 #
 function Invoke-Cmd{
 	param($cmd, $arguments)
 
 	$global:StdOut = ""
 	$global:StdErr = ""
 
 	$pStart = new-object System.Diagnostics.ProcessStartInfo
 	$pStart.Filename = $cmd
 	$pStart.Arguments = $arguments
 	$pStart.UseShellExecute = $false
 	$pStart.ErrorDialog = $false
 	$pStart.CreateNoWindow = $True
 	$pStart.RedirectStandardOutput = $true
 	$pStart.RedirectStandardError = $true
 	$myProcess = [System.Diagnostics.Process]::Start($pStart)
 
 	$myOutput = $myProcess.StandardOutput
 	$myErrOutput = $myProcess.StandardError
 	$global:StdOut = $myOutput.ReadToEnd()
 	$global:StdErr = $myErrOutput.ReadToEnd()
 
 	$myProcess.WaitForExit()
 
 	$myProcess.ExitCode
 }
 
 #
 function Import-VCBImage{
 	param($vmName, $portgroup) 
 	if((Get-Item -path ($I2VImageDir + "\*") | Where-Object {$_.Name -eq $vmName} | Measure-Object).Count -ne 1){
 		Write-Host "Snapshot directory not found for " $vmName 
 		return 
 	} 
 	$selectESX = "" 
 	$selectDS = "" 
 	$selectFree = 0 
 	Get-Datacenter $tgtDatacenter | Get-VMHost | % { 
 		$tmpESX = $_.Name 
 		$_ | Get-Datastore | % { 
 			if($_.FreeSpaceMb -gt $selectFree){ 
 				$selectESX = $tmpESX 
 				$selectDS = $_.Name 
 				$selectFree = $_.FreeSpaceMb 
 			} 
 		} 
 	} 
 	if(((Get-Item -path ($I2VImageDir + "\" + $vmName + "\*") | Measure-Object -property Length -sum).Sum / 1mb) -gt $selectFree){
 		Write-Host "Not enough free disk space on" $selectFree 
 		return 
 	} 
 	$vmxName = (Get-Item -path ($I2VImageDir + "\" + $vmName + "\*") | Where-Object {$_.Name -like "*.vmx"}).Name
 	$p2v.p2v.source.hostedSpec.path = "\\" + $I2Vhost + "\" + $I2VShare + "\" + $vmName + "\" + $vmxName
 	$p2v.p2v.source.hostedSpec.networkUsername = $I2Vuser
 	$p2v.p2v.source.hostedSpec.networkPassword = $I2Vpassword 
 	$p2v.p2v.dest.managedSpec.creds.username = $I2Vuser 
 	$p2v.p2v.dest.managedSpec.creds.password = $I2Vpassword 
 	$p2v.p2v.dest.managedSpec.datastore = $selectDS 
 	$p2v.p2v.dest.managedSpec.host = $selectESX 
 	$p2v.p2v.dest.managedSpec.folder = "" 
 	$p2v.p2v.dest.managedSpec.resourcePool = "" 
 	$V2VvmName = $vmName + "-" + $tgtDatacenter + "-" + (Get-Date -format yyyyMMdd-HHmmss)
 	$p2v.p2v.dest.managedSpec.vmName = $V2VvmName 
 	$p2v.p2v.dest.managedSpec.creds.host = $I2Vhost 
 	$p2v.p2v.dest.managedSpec.creds.username = $I2Vuser 
 	$p2v.p2v.dest.managedSpec.creds.password = $I2Vpassword 
 	$NIC = $p2v.CreateElement("nicMapping") 
 	$network = $p2v.CreateAttribute("network") 
 	$network.psbase.Value = $portgroup
 	$dummy = $NIC.SetAttributeNode($network) 
 	$p2v.p2v.importParams["nicMappings"].AppendChild($NIC) 
 	$XMLfile = $I2VImageDir + "\" + "I2V-" + $vmName + ".xml" 
 	$p2v.Save($XMLfile) 
 	if((Get-Service -name $ConvService).Status -eq "Stopped"){ 
 		Start-Service -name $ConvService 
 	} 
 	$myarg = "--vcHost " + $I2Vhost + " --jobExec " + $XMLfile + " --vcCred " + $I2Vuser + ":" + $I2Vpassword 
 	$mycmd = $ConvProgDir + "\" + $I2Vprog 
 	$rc = Invoke-Cmd $mycmd $myarg 
 
 	if($rc -eq 0){ 
 		foreach($vm in (Get-Datacenter $tgtDatacenter | Get-VM ($vmName + "-" + $tgtDatacenter + "*") | where {$_.Name -ne $V2VvmName})){ 
 			$vm | Remove-VM - DeleteFromDisk:$true - Confirm:$false 
 		} 
 	} 
 } 
 
`

