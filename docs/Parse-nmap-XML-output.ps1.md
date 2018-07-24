---
Author: jason fossen
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 1179
Published Date: 2010-06-27t07
Archived Date: 2017-04-30t13
---

# parse nmap xml output - 

## Description

a powershell script into which one or more nmap xml output file objects can be piped, then the script emits synthetic objects representing port-scanned hosts from the xml file(s).  get windows and linux versions of the nmap scanner for free from http

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ####################################################################################
 ####################################################################################
 
 
 if ($args -ne $null) { 
     "`nThis script takes no arguments, please pipe one or more files into it."
     "Example: dir *.xml | .\parse-nmap.ps1 | export-csv -path c:\file.csv`n"
     exit 
 }
 
 $ShowProgress = $true
 
 
 
 ForEach ($file in $input) {
     If ($ShowProgress) { [Console]::Error.WriteLine("[" + (get-date).ToLongTimeString() + "] Starting $file" ) }
 
     $xmldoc = new-object System.XML.XMLdocument
     $xmldoc.Load($file)
 
     $xmldoc.nmaprun.host | foreach-object { 
 
         $entry = ($entry = " " | select-object FQDN, HostName, Status, IPv4, IPv6, MAC, Ports, OS) 
 
         $entry.Status = $hostnode.status.state.Trim() 
         if ($entry.Status.length -eq 0) { $entry.Status = "<no-status>" }
 
         $hostnode.hostnames | foreach-object { $entry.FQDN += $_.hostname.name + " " } 
         $entry.FQDN = $entry.FQDN.Trim()
         if ($entry.FQDN.Length -eq 0) { $entry.FQDN = "<no-fullname>" }
 
         if ($entry.FQDN.Contains(".")) { $entry.HostName = $entry.FQDN.Substring(0,$entry.FQDN.IndexOf(".")) }
         elseif ($entry.FQDN -eq "<no-fullname>") { $entry.HostName = "<no-hostname>" }
         else { $entry.HostName = $entry.FQDN }
 
         $hostnode.address | foreach-object {
             if ($_.addrtype -eq "ipv4") { $entry.IPv4 += $_.addr + " "}
             if ($_.addrtype -eq "ipv6") { $entry.IPv6 += $_.addr + " "}
             if ($_.addrtype -eq "mac")  { $entry.MAC  += $_.addr + " "}
         }        
         if ($entry.IPv4 -eq $null) { $entry.IPv4 = "<no-ipv4>" } else { $entry.IPv4 = $entry.IPv4.Trim()}
         if ($entry.IPv6 -eq $null) { $entry.IPv6 = "<no-ipv6>" } else { $entry.IPv6 = $entry.IPv6.Trim()}
         if ($entry.MAC  -eq $null) { $entry.MAC  = "<no-mac>" }  else { $entry.MAC  = $entry.MAC.Trim() }
 
 
         if ($hostnode.ports.port -eq $null) { $entry.Ports = "<no-ports>" } 
         else 
         {
             $hostnode.ports.port | foreach-object {
                 If ($_.service.name -eq $null) { $service = "unknown" } else { $service = $_.service.name } 
                 $entry.Ports += $_.state.state + ":" + $_.protocol + ":" + $_.portid + ":" + $service + " " 
             }
             $entry.Ports = $entry.Ports.Trim()
         }
 
 
         $hostnode.os.osmatch | foreach-object {$entry.OS += $_.name + " <" + ([String] $_.accuracy) + "%-accuracy> "} 
         $entry.OS = $entry.OS.Trim()
         if ($entry.OS -eq "<%-accuracy>") { $entry.OS = "<no-os>" }
 
 
         $entry
     }
 
 If ($ShowProgress) { [Console]::Error.WriteLine("[" + (get-date).ToLongTimeString() + "] Finished $file, processed $i entries." ) }
 }
`

