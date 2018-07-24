---
Author: nathan linley
Publisher: 
Copyright: 
Email: 
Version: 4.4
Encoding: ascii
License: cc0
PoshCode ID: 5632
Published Date: 2016-12-07t06
Archived Date: 2016-02-02t12
---

# dell hardware checker - 

## Description

this script runs a variety of checks on server hardware to get status or basic information.  it can check memory, perc events, firmware versions, gets service tags, shows summary hardware status, retrieves esm log, and can auto load the omsa webpage.  this is a basic overview of some dell wmi related namespace operations which can be easily extended as needed.  to run the script, provide the name of the function you want to call and the remote machine name to check.  note

## Comments



## Usage



## TODO



## function

`load-omsa`

## Code

`#
 #
 function load-omsa([string]$server) {
 
 	$omsaie = new-object -com InternetExplorer.Application
 	if (($server -eq $null) -or ($server -eq "")) {
 		write-host -foregroundcolor "yellow"  "Usage: omsa servername"
 		write-host -foregroundcolor "yellow"  "     Enter servername to open OMSA connection to server"
 		write-host -foregroundcolor "yellow"  ""
 		exit
 	}
 
 	$portopen = tcpt $server 1311
 	if ($portopen -eq $false) {
 		write-host -foregroundcolor "red" "OMSA Port not accessible, may be behind firewall or service not running"
 		exit
 	}
 
 
 	
 	$omsaie.visible = 1
 	$url = "https://" + $server + ":1311//servlet/Login?omacmd=getlogin&amp;page=Login"
 	$omsaie.navigate($url)
 	sleep 5
 	if ($omsaie.document -ne $null) {	
 		if ($omsaie.document.getElementById("overridelink") -ne $null) {
 			$omsaie.document.getElementById("overridelink").click()	
 			
 		} 
 	}
 
 
 }
 
 function tcpt ([string]$serv, [string]$p) {
 	 $error.clear()
 	 trap {continue;}
 	 $conn = new-object system.net.sockets.tcpclient($serv,$p)
 	 if ($error) { return $false } else { return $true}
 }
 
 
 
 Function memory ([string]$server) {
 	$errfound = $false
 	if ($server -eq $null) {
 		write-host -foregroundcolor "yellow"  "Usage memory servername"
 		write-host -foregroundcolor "yellow"  "   Enter servername to see DIMM status"
 		exit 
 	}
 
 	$server = &namescrub $server
 
 	$a = Get-WMIObject -Namespace root\CIMv2\Dell -computername $server -Class CIM_PhysicalMemory -erroraction silentlycontinue -errorvariable err
 
 	if (-not($?)) {
 		omsaerr
 		exit
 	}
 
 	if ($a -eq $null) {
 		write-host -foregroundcolor "red" "Could not get memory information on this machine"
 		&wantomsa
 		write-host
 		exit 
 	} else {
 		$b = $a |select-object Name,Capacity,Status
 		echo "Location           Capacity      Status"
 		write-host
 		foreach ($item in $b) {
 			$item.capacity = $item.capacity / 1073741824
 			$itemstring = [string]$item.capacity + " GB"
 			write-host $item.Name "       "   $itemstring  "       "   $item.status
 	
 		}
 		foreach ($item in $b) {
 			if ($item.Status -eq "Error") {
 				$errfound = $true
 			}
 		}
 
 		write-host
 		if ($errfound) {
 			&amdb $server
 		}
 
 		exit 
 	}
 }
 
 Function hwlog ([string]$server) {
 	if ($server -eq $null) {
 		write-host -foregroundcolor "yellow"  "Usage hwlog servername"
 		write-host -foregroundcolor "yellow"  "    Enter servername to pull OMSA hardware event log"
 		write-host -foregroundcolor "yellow"  ""
 		exit
 	}
 
 	$server = &namescrub $server
 
 	$a = Get-WMIOBject -Namespace root\CIMv2\DELL -computername $server -Class DELL_EsmLog -erroraction silentlycontinue -errorvariable err
 	
 	if (-not($?)) {
 		omsaerr
 		exit
 	}
 
 	if ($a -eq $null) {
 		write-host -foregroundcolor "red" "Could not get omsa log for this machine"
 		write-host
 		exit
 	} else {
 
 		$b = $a |select-object EventTime,LogRecord
 		foreach ($item in $b) {
 			if ($item.EventTime -ne $null) {
 				$item.EventTime = &converttime $item.EventTime
 			} else {
 				
 			}
 		}
 
 		$b |format-table -wrap -autosize |more
 
 		write-host
 	}
 }
 
 
 Function perc ([string]$server) {
 	if ($server -eq $null) {
 		write-host -foregroundcolor "yellow"  "Usage perc servername"
 		write-host -foregroundcolor "yellow"  "    Enter servername to get Perc events"
 		write-host
 		exit
 	}
 
 	$server = &namescrub $server
 
 	$a = Get-WmiObject -Class Win32_NTLogEvent -filter "Logfile='Application' AND SourceName='VxSvc_PercPro' AND Type='Warning'" -computer $server -erroraction silentlycontinue -errorvariable err
 
 	if ($a -eq $null) {
 		write-host -foregroundcolor "red" "Could not get event log perc events"
 		&wantomsa
 		write-host
 		exit
 	} else {
 		$n = 6
 		$b = $a |select-object TimeGenerated,EventCode,Message
 		foreach ($item in $b) {
 			if ($item.TimeGenerated -ne $null) {
 				$item.TimeGenerated = &converttime $item.TimeGenerated
 				write-host $item.TimeGenerated   $item.EventCode   $item.Message
 			} else {
 				write-host "No time given"   $item.LogRecord
 			}
 			$n = $n - 1
 			if ($n -eq 0) { write-host; exit}
 		}
 		#$b | format-list  |more		
 		
 
 	}
 }
 
 Function hwstatus ([string]$server) {
 	if ($server -eq $null) {
 		write-host -foregroundcolor "yellow"  "Usage hwstatus servername"
 		write-host -foregroundcolor "yellow"  "   Enter servername to get status of chassis"
 		write-host
 		exit
 	}
 
 	$server = &namescrub $server
 
 	$a = Get-WMIObject -Namespace root\CIMv2\Dell -computername $server -Class DELL_Chassis -erroraction silentlycontinue -errorvariable err
 
 	if (-not($?)) {
 		omsaerr
 		exit
 	}	
 
 	if ($a -eq $null) {
 		write-host -foregroundcolor "red" "Could not get chassis status on this machine"
 		write-host
 		exit
 	} else {
 
 		$b = $a |select-object Model,SerialNumber,Status,ProcStatus,PsStatus,TempStatus,VoltStatus,FanStatus
 		$b
 	}
 
 }
 
 
 function omsaerr {
     
 	$exception = $error[0].Exception.ErrorCode
 	if ($exception -eq "InvalidNamespace") {
 		write-host "OMSA version not high enough to query machine"  -foregroundcolor "red"
 		
 	} elseif ($error[0].CategoryInfo.Reason -eq "UnauthorizedAccessException") {
 		write-host "Access is Denied" -foregroundcolor "red"
 	} elseif ($error[0].Exception.ErrorCode -eq -2147023174) {	
 		write-host "Can't connect to RPC service on machine, if OS is windows, server might be down or behind a firewall. " -foregroundcolor "red"
 	} else {
 		write-host "OMSA connection failed" -foregroundcolor "red"
 	
 	}
 		write-host
 		$error.clear()
 		&wantomsa
 }
 
 function firmware ([string]$server) {
 	if (($server -eq $null) -or ($server -eq "")) {
 		write-host -foregroundcolor "yellow"  "Usage:  Firmware servername"
 		write-host -foregroundcolor "yellow"  "    Enter servername to get all firmware information on the dell components"
 		write-host
 		exit
 	}
 
 	$server = &namescrub $server
 
 	$a = Get-WMIObject -Namespace root\CIMv2\Dell -computername $server -Class DELL_Firmware -erroraction silentlycontinue -errorvariable err
 
 	if (-not($?)) {
 		omsaerr
 		exit
 	}	
 
 	if ($a -eq $null) {
 		write-host -foregroundcolor "red" "Could not get firmware information for this machine (OMSA needs to be >=4.4)"
 		&wantomsa
 		write-host
 		exit
 	} else {
 
 		$b = $a |select-object Name,SoftwareElementID
 		$b
 	}
 
 }
 
 
 Function wantomsa() {
 	
 	$omsatest = tcpt $server 1311
 	if ($omsatest) {
 		write-host
 		echo "Would you like to run OMSA to check further? (y/n)"
 		$key = $host.ui.RawUI.ReadKey(12)
 		if (($key.Character -eq 'y') -or ($key.Character -eq 'Y')) {
 			&load-omsa $server
 		}
 
 	}
 	write-host
 }
 
 Function gettag([string]$server) {
 
 	$server = &namescrub $server
 
 	if ($server -eq "") {
 		write-host -foregroundcolor "yellow"  "Usage: gettag servername"
 		write-host -foregroundcolor "yellow"  "    Enter the name of the server that you would like to get"
 		write-host -foregroundcolor "yellow"  "    the service tag for"
 		exit
 	}
 
 	write-host
 	$a = Get-WmiObject Win32_BIOS -computername $server -erroraction silentlycontinue -errorvariable err
 	if ($a -eq $null) { 
 		write-host -foregroundcolor "red" "Server not found or can't connect, could not get tag number"
 		&wantomsa
 		exit
 	} 
 	else {
 		$a = $a | select-object SerialNumber
 		$b = $a.serialnumber
 		echo "Service tag for $server is $b"
 		write-host
 	}
 }
 
 Function converttime ([string]$timestr) {
 	$formattime = $timestr.Substring(0,4) + " " + $timestr.Substring(4,2) + "M " + $timestr.Substring(6,2) + "D " + $timestr.Substring(8,2) + ":" + $timestr.Substring(10,2)
 	return $formattime
 }
 
 $server = $args[1]
 $function = $args[0]
 
 if ($function -eq $null -or $server -match "\?") {
 	Write-Host -ForegroundColor "yellow" "Usage:  Dell-hwchecks-wmi.ps1 <function> [servername]"
 	Write-Host -ForegroundColor "yellow" "     Functions:"
 	Write-Host -ForegroundColor "yellow" "        hwlog     -  Prints ESM log"
 	write-host -ForegroundColor "yellow" "        load-omsa -  Open OMSA web interface on remote machine"
 	Write-Host -ForegroundColor "yellow" "        hwstatus  -  Chassis component health check"
 	Write-Host -ForegroundColor "yellow" "        gettag    -  Print service tag"
 	Write-Host -ForegroundColor "yellow" "        firmware  -  List firmware versions"
 	Write-Host -ForegroundColor "yellow" "        perc      -  Search event log for perc events"
 	Write-Host -ForegroundColor "yellow" "        memory   -  List memory modules and status"
 	exit
 }
 if ($server -eq $null -or $server -eq "") { $server = "locahost" }
 
 &$function $server
`

