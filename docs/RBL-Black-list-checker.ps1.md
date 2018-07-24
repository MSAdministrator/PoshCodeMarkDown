---
Author: orbgod
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5867
Published Date: 2016-05-22t11
Archived Date: 2016-09-07t04
---

# rbl black list checker - 

## Description

this script will check a list of ip’s versus the most common rbl’s online – it will mail results of each ip with a list of the blacklists it was found on. if you run it during the 9am -10am hours it will send negitive emails also for a sanity check –

## Comments



## Usage



## TODO



## script

`get-blacklist`

## Code

`#
 #
 ####################################################################################################
 ####################################################################################################
 
 function Get-Blacklist
 {
 <#
 .SYNOPSIS
 	checks IP's against most common Blacklists
 .DESCRIPTION
 	This script will use a list of common RBL sites and attempt to resolve a list of IP addresses to each RBL, if found it will send an email to the defined admin with the BL list info and the IP.
 .LINK
 	http://www.poshcode.org/
 .EXAMPLE
 	foreach ($NAT in $ExternalNAT)
 	{
 	$address = $NAT.Octet
 	Get-blacklist $address
 	}
 .PARAMETER IP
 	the IP address (IPV4 only!) to be checked
 .NOTES
  	History:
 		v 1.0 - Set up, BL lists to check, and Function Creation.
 #>
 param($IP)
 
 $adminemail = 'admin@yourdomain.com'
 $fromemail = 'BLACKLIST-CHECK@yourdomain.com'
 $mailserver = 'mailrelay.yourdomain.com'
 $subjectfail = 'RBL listing detected! '+$IP
 $subjectpass = 'RBL not listed :)'
 
 $reversedIP = ($IP -split '\.')[3..0] -join '.'
 
 $blacklistServers = @(
 'b.barracudacentral.org'
 'bl.spamcannibal.org'
 'blacklist.woody.ch'
 'cdl.anti-spam.org.cn'
 'db.wpbl.info'
 'dnsbl-3.uceprotect.net'
 'dnsbl.inps.de'
 'drone.abuse.ch'
 'dul.dnsbl.sorbs.net'
 'dynip.rothen.com'
 'ips.backscatterer.org'
 'misc.dnsbl.sorbs.net'
 'omrs.dnsbl.net.au'
 'osrs.dnsbl.net.au'
 'pbl.spamhaus.org'
 'proxy.bl.gweep.ca'
 'rbl.interserver.net'
 'relays.bl.gweep.ca'
 'residential.block.transip.nl'
 'sbl.spamhaus.org'
 'socks.dnsbl.sorbs.net'
 'spam.rbl.msrbl.net'
 'spamrbl.imp.ch'
 'tor.dnsbl.sectoor.de'
 'ubl.unsubscore.com'
 'virus.rbl.msrbl.net'
 'xbl.spamhaus.org'
 'bl.deadbeef.com'
 'bl.spamcop.net'
 'bogons.cymru.com'
 'combined.abuse.ch'
 'dnsbl-1.uceprotect.net'
 'dnsbl.njabl.org'
 'dul.ru'
 'http.dnsbl.sorbs.net'
 'ix.dnsbl.manitu.net'
 'noptr.spamrats.com'
 'orvedb.aupads.org'
 'owfs.dnsbl.net.au'
 'phishing.rbl.msrbl.net'
 'proxy.block.transip.nl'
 'rbl.megarbl.net'
 'relays.bl.kundenserver.de'
 'ricn.dnsbl.net.au'
 'short.rbl.jp'
 'spam.abuse.ch'
 'spam.spamrats.com'
 't3direct.dnsbl.net.au'
 'torserver.tor.dnsbl.sectoor.de'
 'virbl.bit.nl'
 'web.dnsbl.sorbs.net'
 'zen.spamhaus.org'
 'bl.emailbasura.org'
 'blackholes.five-ten-sg.com'
 'cbl.abuseat.org'
 'combined.rbl.msrbl.net'
 'dnsbl-2.uceprotect.net'
 'dnsbl.cyberlogic.net'
 'dnsbl.sorbs.net'
 'duinv.aupads.org'
 'dyna.spamrats.com'
 'images.rbl.msrbl.net'
 'korea.services.net'
 'ohps.dnsbl.net.au'
 'osps.dnsbl.net.au'
 'owps.dnsbl.net.au'
 'probes.dnsbl.net.au'
 'psbl.surriel.com'
 'rdts.dnsbl.net.au'
 'relays.nether.net'
 'rmst.dnsbl.net.au'
 'smtp.dnsbl.sorbs.net'
 'spam.dnsbl.sorbs.net'
 'spamlist.or.kr'
 'ubl.lashback.com'
 'virus.rbl.jp'
 'wormrbl.imp.ch'
 'zombie.dnsbl.sorbs.net'
 )
 
 $blacklistedOn = @()
 
 foreach ($server in $blacklistServers)
 {
     $fqdn = "$reversedIP.$server"
     try
     {
         $null = [System.Net.Dns]::GetHostEntry($fqdn)
         $blacklistedOn += $server
     }
     catch { }
 }
 
 if ($blacklistedOn.Count -gt 0)
 {
     send-mailmessage -Priority High -from $fromemail -to $adminemail -SMTPServer $mailserver -body "$IP is blacklisted on the following servers: $($blacklistedOn -join ', ')" -Subject $subjectfail
   }
 else
 {
     Write-Host "$IP is not currently blacklisted on any server."
 
  if ((Get-Date).Hour -eq 10)
     {
     send-mailmessage -from $fromemail -to $adminemail -SMTPServer $mailserver -body "$IP is not blacklisted" -Subject $subjectpass
 }
 }
 }
`

