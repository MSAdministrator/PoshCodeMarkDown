---
Author: kenneth c mazie
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 4014
Published Date: 2015-03-13t19
Archived Date: 2015-05-14t10
---

# esx host ntp settings - 

## Description

this script will synchronize the ntp settings across all esx hosts in all clusters in a specified virtual center instance.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#======================================================================================
          File Name : ESX-NTP.ps1
    Original Author : Kenneth C. Mazie
                    :
        Description : This is a VMware PowerCLI script for synchronizing NTP settings across
                    : all ESX hosts in a Virtual Center instance.  It grabs all ESX hosts and
                    : cycles through them setting the same NTP settings on each.  It then
                    : starts the service and sets it to start automatically.
                    :
                    : The individual commands were found on a blog post from Anders Mikkelson.
                    : I simply combined them into a functioning script so credit goes to him.
                    : See http://www.amikkelsen.com/?p=458 for the original posting.
                    :
                    : Tested on ESXi 5.1 and 5.0
                    :
                    : Be sure to edit all sections to suit your needs before executing.  
                    :
                    : As written the script will prompt for a Virtual Center instance to process
                    : and a user/password to connect as.  It also prompts for the ESX host user ID.
                    : Hard code these if the prompts become annoying.
                    :
              Notes : Requires VMware PowerShell PowerCLI extentions.
                    :
           Warnings : 
                    :  
                    :
     Last Update by : Kenneth C. Mazie (kcmjr AT kcmjr.com to report issues)
    Version History : v1.0 - 03-13-12 - Original
     Change History : v1.1 - 03-13-12 - Corrected 2 typos 
                    :
 =======================================================================================#>
 
 clear-host
 $out = Get-PSSnapin | Where-Object {$_.Name -like "vmware.vimautomation.core"};if ($out -eq $null) {Add-PSSnapin vmware.vimautomation.core}
 
 $vSphere = read-host "Process Which vSphere Instance?"
 $vUser = read-host "vSphere User ID?"
 $vPass = ""
 $eUser = read-host "ESX User ID?"
 $ePass = "password"
 $NTPServers = ("time.nist.gov","time-nw.nist.gov","us.pool.ntp.org")    #--[ Change these to whatever NTP servers you use ]--
 
 if ($vSphere -eq $null){break} 
 Disconnect-VIServer -confirm:$false
 Connect-VIServer -Server $vSphere -User $vUser #-Password $vPass
 
 $ESXhosts = get-vmhost
 
 foreach ($ESX in $ESXHosts)  
   {
 Write-Host "Target = $ESX"
 
 Connect-VIServer -Server $ESX -User $eUser #-Password $ePass
   
 $NTPList = Get-VMHostNtpServer -VMHost $ESX
 Remove-VMHostNtpServer -VMHost $ESX -NtpServer $NTPList -Confirm:$false 
 
 Add-VMHostNtpServer -VMHost $ESX -NtpServer $NTPServers -Confirm:$false
   
 Set-VMHostService -HostService (Get-VMHostservice -VMHost (Get-VMHost $ESXhost) | Where-Object {$_.key -eq "ntpd"}) -Policy "Automatic" 
 
 Get-VmhostFirewallException -VMHost $ESX -Name "NTP Client" | Set-VMHostFirewallException -enabled:$true  
 
 $ntpd = Get-VMHostService -VMHost $ESX | where {$_.Key -eq 'ntpd'}
 Restart-VMHostService $ntpd -Confirm:$false 
 
 Disconnect-viserver -server $ESX -confirm:$false
   
   }
 
 Write-Host "--- COMPLETED ---"
`

