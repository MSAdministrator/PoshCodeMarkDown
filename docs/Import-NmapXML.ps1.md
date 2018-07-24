---
Author: jason fossen
Publisher: 
Copyright: 
Email: 
Version: 3.6.1-jwg1
Encoding: ascii
License: cc0
PoshCode ID: 2596
Published Date: 2012-04-01t16
Archived Date: 2016-03-28t17
---

# import-nmapxml - 

## Description

this is a powershell v2 module that takes nmap xml output and formats it into custom powershell objects, allowing you to manipulate nmap output data in powershell. it operates similarly to import-csv.

## Comments



## Usage



## TODO



## function

`import-nmapxml`

## Code

`#
 #
 function Import-NmapXML 
 {
 	####################################################################################
 	#.Synopsis 
 	#
 	#.Description 
 	#
 	#.Parameter Path  
 	#
 	#.Parameter OutputDelimiter
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
 	#.Example 
 	#
 	#
 	#.Notes 
 	####################################################################################
     [CmdletBinding()]
     
 	param (
         [Parameter(Mandatory=$true,ValueFromPipeline=$true)]$Path, 
         [String] $OutputDelimiter = "`n", 
         [Switch] $RunStatsOnly,
         [Switch] $ShowProgress
     )
 	
 	if ($Path -match "/\?|/help|-h|-help|--h|--help") 
 	{ 
 		"`nPurpose: Process nmap XML output files (www.nmap.org).`n"
 		"Example: Import-NmapXML scanfile.xml"
         "Example: Import-NmapXML *.xml -runstatsonly `n"
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
 			$stat = ($stat = " " | select-object FilePath,FileName,Scanner,Profile,ProfileName,Hint,ScanName,Arguments,Options,NmapVersion,XmlOutputVersion,StartTime,FinishedTime,ElapsedSeconds,ScanTypes,TcpPorts,UdpPorts,IpProtocols,SctpPorts,VerboseLevel,DebuggingLevel,HostsUp,HostsDown,HostsTotal)
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
             
             $xmldoc.nmaprun.scaninfo | foreach {
                 $stat.ScanTypes += $_.type + " "
 
                 if ($services.contains("-"))
                 {
                     $array = $($services.replace("-","..")).Split(",")
                     $temp  = @($array | where { $_ -notlike "*..*" })  
                     $array | where { $_ -like "*..*" } | foreach { invoke-expression "$_" } | foreach { $temp += $_ } 
                     $temp = [Int32[]] $temp | sort 
                     $services = [String]::Join(",",$temp) 
                 } 
                     
                 switch ($_.protocol)
                 {
                     "tcp"  { $stat.TcpPorts  = $services ; break }
                     "udp"  { $stat.UdpPorts  = $services ; break }
                     "ip"   { $stat.IpProtocols = $services ; break }
                     "sctp" { $stat.SctpPorts = $services ; break }
                 }
             } 
             
             $stat.ScanTypes = $($stat.ScanTypes).Trim()
             
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
 		
 			$entry = ($entry = " " | select-object FQDN, HostName, Status, IPv4, IPv6, MAC, Ports, Services, OS, Script) 
 
 			$entry.Status = $hostnode.status.state.Trim() 
 			if ($entry.Status.length -lt 2) { $entry.Status = "<no-status>" }
 
             $hostnode.hostnames.hostname | foreach-object { $entry.FQDN += $_.name + " " } 
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
 
 
 			if ($hostnode.ports.port -eq $null) { $entry.Ports = "<no-ports>" ; $entry.Services = "<no-services>" } 
 			else 
 			{
 				$hostnode.ports.port | foreach-object {
 					if ($_.service.name -eq $null) { $service = "unknown" } else { $service = $_.service.name } 
 					$entry.Ports += $_.state.state + ":" + $_.protocol + ":" + $_.portid + ":" + $service + $OutputDelimiter 
                     if ($_.state.state -like "open*" -and ($_.service.tunnel.length -gt 2 -or $_.service.product.length -gt 2 -or $_.service.proto.length -gt 2)) { $entry.Services += $_.protocol + ":" + $_.portid + ":" + $service + ":" + ($_.service.product + " " + $_.service.version + " " + $_.service.tunnel + " " + $_.service.proto + " " + $_.service.rpcnum).Trim() + " <" + ([Int] $_.service.conf * 10) + "%-confidence>$OutputDelimiter" }
 				}
 				$entry.Ports = $entry.Ports.Trim()
                 if ($entry.Services -eq $null) { $entry.Services = "<no-services>" } else { $entry.Services = $entry.Services.Trim() }
 			}
 
 
 			$hostnode.os.osmatch | foreach-object {$entry.OS += $_.name + " <" + ([String] $_.accuracy) + "%-accuracy>$OutputDelimiter"} 
             $hostnode.os.osclass | foreach-object {$entry.OS += $_.type + " " + $_.vendor + " " + $_.osfamily + " " + $_.osgen + " <" + ([String] $_.accuracy) + "%-accuracy>$OutputDelimiter"}  
             $entry.OS = $entry.OS.Replace("  "," ")
 			$entry.OS = $entry.OS.Trim()
 			if ($entry.OS.length -lt 16) { $entry.OS = "<no-os>" }
 
             
             $hostnode.ports.port | foreach-object {
                 if ($_.script -ne $null) { 
                     $entry.Script += "<PortScript id=""" + $_.script.id + """>$OutputDelimiter" + ($_.script.output -replace "`n","$OutputDelimiter") + "$OutputDelimiter</PortScript> $OutputDelimiter $OutputDelimiter" 
                 }
             } 
             
             if ($hostnode.hostscript -ne $null) {
                 $hostnode.hostscript.script | foreach-object {
                     $entry.Script += '<HostScript id="' + $_.id + '">' + $OutputDelimiter + ($_.output.replace("`n","$OutputDelimiter")) + "$OutputDelimiter</HostScript> $OutputDelimiter $OutputDelimiter" 
                 }
             }
             
             if ($entry.Script -eq $null) { $entry.Script = "<no-script>" } 
     
     
 			$entry
 		}
 
 		If ($ShowProgress) { [Console]::Error.WriteLine("[" + (get-date).ToLongTimeString() + "] Finished $file, processed $i entries." ) }
 	}
 }
`

