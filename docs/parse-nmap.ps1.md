---
Author: jason fossen
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 1219
Published Date: 
Archived Date: 2009-07-22t10
---

#  - 

## Description

get a sample nmap xml file to play with and see some examples of using the script at https

## Comments



## Usage



## TODO



## function

`parse-nmap`

## Code

`#
 #
 Param ($Path, [Switch] $RunStatsOnly, [Switch] $ShowProgress)
 
 function parse-nmap 
 {
 	####################################################################################
 	#.Synopsis 
 	#
 	#.Description 
 	#
 	#.Parameter Path  
 	#
 	#.Parameter RunStatsOnly
 	#
 	#.Parameter ShowProgress
 	#
 	#.Example 
 	#
 	#.Example 
 	#
 	#.Example 
 	#
 	#.Example 
 	#
 	#
 	#.Notes 
 	####################################################################################
 
 	param ($Path, [Switch] $RunStatsOnly, [Switch] $ShowProgress)
 	
 	if ($Path -match "/\?|/help|-h|-help|--h|--help") 
 	{ 
 		"`nPurpose: Process nmap XML output files (www.nmap.org)."
 		"Example: dir *.xml | .\parse-nmap.ps1 `n"
 		exit 
 	}
 
 	if ($Path -eq $null) {$Path = @(); $input | foreach { $Path += $_ } } 
 	$1970 = [DateTime] "01 Jan 1970 01:00:00 GMT"
 
 	if ($RunStatsOnly)
 	{
 		ForEach ($file in $Path) 
 		{
 			$xmldoc = new-object System.XML.XMLdocument
 			$xmldoc.Load($file)
 			$stat = ($stat = " " | select-object FilePath,FileName,Scanner,Profile,ProfileName,Hint,ScanName,Arguments,Options,NmapVersion,XmlOutputVersion,StartTime,FinishedTime,ElapsedSeconds,ScanType,Protocol,NumberOfServices,Services,VerboseLevel,DebuggingLevel,HostsUp,HostsDown,HostsTotal)
 			$stat.FilePath = $file.fullname
 			$stat.FileName = $file.name
 			$stat.Scanner = $xmldoc.nmaprun.scanner
 			$stat.Profile = $xmldoc.nmaprun.profile
 			$stat.ProfileName = $xmldoc.nmaprun.profile_name
 			$stat.Hint = $xmldoc.nmaprun.hint
 			$stat.ScanName = $xmldoc.nmaprun.scan_name
 			$stat.Arguments = $xmldoc.nmaprun.args
 			$stat.Options = $xmldoc.nmaprun.options
 			$stat.NmapVersion = $xmldoc.nmaprun.version
 			$stat.XmlOutputVersion = $xmldoc.nmaprun.xmloutputversion
 			$stat.StartTime = $1970.AddSeconds($xmldoc.nmaprun.start) 	
 			$stat.FinishedTime = $1970.AddSeconds($xmldoc.nmaprun.runstats.finished.time)
 			$stat.ElapsedSeconds = $xmldoc.nmaprun.runstats.finished.elapsed
 			$stat.ScanType = $xmldoc.nmaprun.scaninfo.type	
 			$stat.Protocol = $xmldoc.nmaprun.scaninfo.protocol		
 			$stat.NumberOfServices = $xmldoc.nmaprun.scaninfo.numservices
 
 			$stat.Services = $($stat.Services.replace("-","..")).Split(",") 
 			$temp  = $stat.Services | where { $_ -notlike "*..*"} | foreach { [Int] $_ } 
 			$stat.Services | where { $_ -like "*..*" } | foreach { invoke-expression "$_" } | foreach { $temp += [Int] $_ } 
 
 			$stat.VerboseLevel = $xmldoc.nmaprun.verbose.level
 			$stat.DebuggingLevel = $xmldoc.nmaprun.debugging.level		
 			$stat.HostsUp = $xmldoc.nmaprun.runstats.hosts.up
 			$stat.HostsDown = $xmldoc.nmaprun.runstats.hosts.down		
 			$stat.HostsTotal = $xmldoc.nmaprun.runstats.hosts.total
 			$stat 			
 		}
 	}
 	
 	ForEach ($file in $Path) {
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
 }
 
 
 
 
 if ($showprogress) {parse-nmap -path $Path -showprogress}
 elseif ($runstatsonly) {parse-nmap -path $Path -runstatsonly}
 else { parse-nmap -path $Path }
`

