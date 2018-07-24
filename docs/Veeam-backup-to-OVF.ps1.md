---
Author: adrianwoodrup
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3970
Published Date: 2013-02-20t15
Archived Date: 2013-03-03t01
---

# veeam backup to ovf - 

## Description

if you want to get an off site backup of your vm’s but dont have a 2nd office and dont want to rely on tapes, then use veeam, amazon aws and a few powershell snapin’s to get your vm’s into the cloud. this solution costs a fraction of a dr budget and the vm’s can be powered on and running in an hour.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $VeeamJob = "JobName"
 $server1 = 'ServerName'
 
 asnp VMware.VimAutomation.Core -ErrorAction SilentlyContinue
 asnp VeeamPSSnapin -ErrorAction SilentlyContinue
 
 $Dest1 = "R:\VMWARE\OVF\$Server1\"
 
 $OVFFile1 = "$($Dest1)$($Server1).ovf"
 
 $VmIRName1 = $Server1 + "_ovf"
 
 $VcName = "vCenterServer"
 $EsxHost = "SingleESXHost"
 $OVFTool = "D:\Program Files\VMware\VMware OVF Tool\ovftool.exe"
 $Date = get-date -Format yy-MM-dd
 $TimeStamp = get-date 
 $Log = "D:\logs\VMWare\" + $Date + ' ' + $Server1 + " OVF to AWS.log"
 $EmailTo = 'Address@Domain.com'
 $EmailFrom = 'Address@Domain.com'
 
 #############################################################################################
 $error.clear()
 Add-content -Path $Log -Value "Backup of Server $Server1 Started at $TimeStamp"
 $ExportObj1 = (Get-VBRBackup -name "$VeeamJob").GetLastOibs() | Where-Object {$_.Name -eq $Server1}
 
 $EsxObj1 = Get-VBRServer -Name "$EsxHost"
 $ResourcePoolObj = Find-VBRResourcePool -Server $EsxObj1 -Name "Resources" 
 Start-VBRInstantRecovery -RestorePoint $ExportObj1 -VMName $VmIRName1 -Server $EsxObj1 -ResourcePool $ResourcePoolObj -PowerUp $false -NICsEnabled $true
 if (!$?) {
 	Add-Content -Path $Log -Value "$server1 InstantRecovery failed with Error (error code:0x00000080): $error"
 	Add-Content -Path $Log -Value "Stopping the backup of $Server1"
 	Break}
 Else{
 	Add-Content -Path $Log -Value "$Server1 InstantRecovery Started successfully..."
 	Connect-VIServer $VCName -User 'ESXAdminUser' -Password 'Password' 
 	if (!$?) {
 		Add-Content -Path $Log -Value "$server1 Connection to vCenter failed with Error (error code:0x00000079): $error"
 		Add-Content -Path $Log -Value "Stopping the backup of $Server1"
 		Break}
 	Else{
 		Add-Content -Path $Log -Value "$Server1 Connection to VI Server was successful..."
 		$VmIR1 = Get-VM -Name "$VmIRName1"
 		$Session = Get-View -Id Sessionmanager
 		$Ticket = $Session.AcquireCloneTicket()
 		& $OVFTool "--name=$Server1" "--noSSLVerify" "--targetType=OVF" "--overwrite" "--I:sourceSessionTicket=$($Ticket)" "vi://$($VcName)?moref=vim.VirtualMachine:$($VmIR1.extensiondata.moref.value)" "$($Dest1)$($Server1).ovf"
 		if (!$?) {
 			Add-Content -Path $Log -Value "$server1 OVF conversion and transfer failed with Error (error code:0x00000078): $error"
 			Add-Content -Path $Log -Value "Stopping the backup of $Server1"
 			Break}
 		Else{
 			Add-Content -Path $Log -Value "$Server1 OVF conversion and transfer completed successfully..."
 			$IR1 = Get-VBRInstantRecovery | where-object {$_.VMName -eq "$VMIRName1"}
 			Stop-VBRInstantRecovery $IR1
 			if (!$?) {
 				Add-Content -Path $Log -Value "$server1 Stop InstantRecovery failed with Error (error code:0x0000077): $error"
 				Add-Content -Path $Log -Value "Stopping the backup of $Server1"
 				Break}
 			Else{
 				Add-Content -Path $Log -Value "$Server1 stopping of the Instant Recovery completed successfully..."
 				Remove-VM $VmIR1 -Confirm:$false 
 				if (!$?) {
 					Add-Content -Path $Log -Value "$server1 Removal of Server_OVF Failed with Error (error code:0x00000076): $error"
 					Add-Content -Path $Log -Value "Stopping the backup of $Server1"
 					Break}
 				Else{
 					Add-Content -Path $Log -Value "$Server1 Removeal of the temp OVF completed successfully..."
 					Disconnect-VIServer $VCName -Confirm:$False
 					if (!$?) {
 						Add-Content -Path $Log -Value "$server1 failed to disconnect from the vCenter server with Error (error code:0x00000075): $error"
 						Add-Content -Path $Log -Value "Stopping backup of $Server1"
 						Break}
 					Else{
 						Add-Content -Path $Log -Value "$Server1 OVF conversion and transfer completed successfully (OK) at $TimeStamp..."
 						
 					}
 				}
 			}
 		}
 	}
 }
 
 #################################################################
 #################################################################
 $Failed = Get-Content $log | Select-String "0x000000" -quiet
 $Success = Get-Content $log | Select-String "(OK)" -quiet
 
 $PSEmailserver = 'EmailServer' 
 If ($Failed){$Subject = $Server1 + ' OVF backup failed'}
 Elseif ($Success) {$subject = $Server1 + ' OVF backup Success'}
 else {$subject = "UNKNOWN OVF backup Result"}
 
 Send-MailMessage -To $EmailTo -From $EmailFrom -Subject $Subject -attachments $log -Body "See the log file for the results of the OVF backup of $Server1"
`

