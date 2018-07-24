---
Author: alberto damiano
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1412
Published Date: 
Archived Date: 2009-10-26t15
---

# check hba status - 

## Description

vmware in a fibre channel enviroment i generate an html report of lun visibility. feedback welcome

## Comments



## Usage



## TODO



## function

`generate-report`

## Code

`#
 #
 ###########################################################################################
 ###########################################################################################
 
 $VCServerName = "YOUR SERVER"
 
 
 $CLUSTERS = @()
 $portvc="443"
 
 
 $VC = Connect-VIServer $VCServerName -ErrorAction Stop -port $portvc
 
 $CLUSTERS = Get-Cluster | Select-Object Name
 
 ForEach ($ClusterName in $CLUSTERS)
 {
 
 	
 	$VMHosts = Get-VMHost -Location $ClusterName.Name | Sort-Object Name
 
 	Function Generate-Report 
 	{
 		Write-Output "<body>"
 		ForEach ($VMHost in $VMHosts)
 		{
 			$Report = @()
 			$ESX = Get-VMHost $VMHost
 			get-vmhoststorage -RescanAllHba -VMHost $ESX > $null
 			$PROPVMHOST = Get-View $ESX.id
 			$storageSystem = Get-View $PROPVMHOST.ConfigManager.StorageSystem
 			$VMHBAs = $PROPVMHOST.Config.StorageDevice.ScsiTopology.Adapter
 			$lista = $storageSystem.StorageDeviceInfo.MultipathInfo.Lun  
 			Write-Output "<table><tr class=""Title""><td colspan=""5"">$($VMhost.Name)</td></tr><tr class="Title"><td>LunID  </td><td>Vmhba  </td><td> WWN SAN </td><td>SCSI Target  </td><td>State  </td></tr>"
 			
 			ForEach ($lun in $lista)
 			{
 				$lun.Path | %{
 					if ($_.Adapter.Contains("FibreChannelHba"))
 					{
 						$OUT = {} | Select Lunid, Vmhba, wwnt, target, state
 						$temp = $lun.id.Split(":")[2]
 						$OUT.Lunid = [Convert]::Todecimal($temp)
 						$OUT.Vmhba = $_.Name.Split(":")[0]
 						$elenco = $storageSystem.StorageDeviceInfo.HostBusAdapter | select Device, PortWorldWideName
 						ForEach ($ele in $elenco)
 						{
 							if ($ele.Device -eq $OUT.Vmhba)
 							{
 								break
 							}
 						}
 						$wwnhex = "{0:x}" -f $ele.PortWorldWideName
 						$OUT.Vmhba = $_.Name.Split(":")[0] + " " + $wwnhex
 						$OUT.wwnt = "{0:x}" -f $_.Transport.PortWorldWideName
 						$OUT.target = $_.Name.Split(":")[1]
 						$OUT.state = $_.PathState
 						$Report += $OUT
 					}
 				}
 			}
 			$Report = $Report | Sort-Object Lunid
 			$old = $Report[1].Lunid
 			$flag = $true
 			Foreach ($rep in $Report) 
 			{
 			    if ($rep.Lunid -ne $old)
 				{
 					$flag = !($flag)
 					$old = $rep.Lunid
 					if ($flag)
 					{
 					}
 					else
 					{
 					}
 				}
 				if ($rep.state -like "dead")
 				{
 					Write-Output "<tr bgcolor=$($bg)><td>$($rep.Lunid)</td><td>$($rep.Vmhba)</td><td>$($rep.wwn)</td><td><center>$($rep.target)</center></td><td class="Error">$($rep.state)</td></tr>" 
 				}
 				else
 				{
 					Write-Output "<tr bgcolor=$($bg)><td>$($rep.Lunid)</td><td>$($rep.Vmhba)</td><td>$($rep.wwnt)</td><td><center>$($rep.target)</center></td><td>$($rep.state)</td></tr>" 
 				}
 			}
 			Write-Output "</table>"
 			Write-Output "<BR>"
 		}
 		Write-Output "</body></html>"
 	}
 
 	
 	Generate-Report >> "c:\temp\SAN_Path_$($ClusterName.Name).html"
 }
`

