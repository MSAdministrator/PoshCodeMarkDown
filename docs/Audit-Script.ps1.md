---
Author: cassibr13
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: utf-8
License: cc0
PoshCode ID: 794
Published Date: 2009-01-09t13
Archived Date: 2017-05-22t04
---

# audit script - 

## Description

507 change $colshares to $colapps

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #####################################################
 #####################################################
 
 param( [string] $auditlist)
 
 if ($auditlist -eq ""){
 	Write-Host "No list specified, using $env:computername"
 	$targets = $env:computername
 }
 else
 {
 	if ((Test-Path $auditlist) -eq $false)
 	{
 		Write-Host "Invalid audit path specified: $auditlist"
 		exit
 	}
 	else
 	{
 		Write-Host "Using Audit list: $auditlist"
 		$Targets = Get-Content $auditlist
 	}
 }
 
 $Date = Get-Date
 Foreach ($Target in $Targets){
 	Write-Output "Collating Detail for $Target"
 	$ComputerSystem = Get-WmiObject -computername $Target Win32_ComputerSystem
 	switch ($ComputerSystem.DomainRole){
 		0 { $ComputerRole = "Standalone Workstation" }
 		1 { $ComputerRole = "Member Workstation" }
 		2 { $ComputerRole = "Standalone Server" }
 		3 { $ComputerRole = "Member Server" }
 		4 { $ComputerRole = "Domain Controller" }
 		5 { $ComputerRole = "Domain Controller" }
 		default { $ComputerRole = "Information not available" }
 	}
 	
 	$OperatingSystems = Get-WmiObject -computername $Target Win32_OperatingSystem
 	$TimeZone = Get-WmiObject -computername $Target Win32_Timezone
 	$Keyboards = Get-WmiObject -computername $Target Win32_Keyboard
 	$SchedTasks = Get-WmiObject -computername $Target Win32_ScheduledJob
 	$BootINI = $OperatingSystems.SystemDrive + "boot.ini"
 	$RecoveryOptions = Get-WmiObject -computername $Target Win32_OSRecoveryConfiguration
 	
 	switch ($ComputerRole){
 		"Member Workstation" { $CompType = "Computer Domain"; break }
 		"Domain Controller" { $CompType = "Computer Domain"; break }
 		"Member Server" { $CompType = "Computer Domain"; break }
 		default { $CompType = "Computer Workgroup"; break }
 	}
 
 	$LBTime=$OperatingSystems.ConvertToDateTime($OperatingSystems.Lastbootuptime)
 	Write-Output "..Hotfix Information"
 	$colQuickFixes = Get-WmiObject Win32_QuickFixEngineering
 	Write-Output "..Logical Disks"
 	$colDisks = Get-WmiObject -ComputerName $Target Win32_LogicalDisk
 	Write-Output "..Network Configuration"
 	$NICCount = 0
 	$colAdapters = Get-WmiObject -ComputerName $Target Win32_NetworkAdapterConfiguration
 	Write-Output "..Local Shares"
 	$colShares = Get-wmiobject -ComputerName $Target Win32_Share
 	Write-Output "..Printers"
 	$colInstalledPrinters =  Get-WmiObject -ComputerName $Target Win32_Printer
 	Write-Output "..Services"
 	$colListOfServices = Get-WmiObject -ComputerName $Target Win32_Service
 	Write-Output "..Regional Options"
 	$ObjKeyboards = Get-WmiObject -ComputerName $Target Win32_Keyboard
 	$keyboardmap = @{
 	"00000402" = "BG" 
 	"00000404" = "CH" 
 	"00000405" = "CZ" 
 	"00000406" = "DK" 
 	"00000407" = "GR" 
 	"00000408" = "GK" 
 	"00000409" = "US" 
 	"0000040A" = "SP" 
 	"0000040B" = "SU" 
 	"0000040C" = "FR" 
 	"0000040E" = "HU" 
 	"0000040F" = "IS" 
 	"00000410" = "IT" 
 	"00000411" = "JP" 
 	"00000412" = "KO" 
 	"00000413" = "NL" 
 	"00000414" = "NO" 
 	"00000415" = "PL" 
 	"00000416" = "BR" 
 	"00000418" = "RO" 
 	"00000419" = "RU" 
 	"0000041A" = "YU" 
 	"0000041B" = "SL" 
 	"0000041C" = "US" 
 	"0000041D" = "SV" 
 	"0000041F" = "TR" 
 	"00000422" = "US" 
 	"00000423" = "US" 
 	"00000424" = "YU" 
 	"00000425" = "ET" 
 	"00000426" = "US" 
 	"00000427" = "US" 
 	"00000804" = "CH" 
 	"00000809" = "UK" 
 	"0000080A" = "LA" 
 	"0000080C" = "BE" 
 	"00000813" = "BE" 
 	"00000816" = "PO" 
 	"00000C0C" = "CF" 
 	"00000C1A" = "US" 
 	"00001009" = "US" 
 	"0000100C" = "SF" 
 	"00001809" = "US" 
 	"00010402" = "US" 
 	"00010405" = "CZ" 
 	"00010407" = "GR" 
 	"00010408" = "GK" 
 	"00010409" = "DV" 
 	"0001040A" = "SP" 
 	"0001040E" = "HU" 
 	"00010410" = "IT" 
 	"00010415" = "PL" 
 	"00010419" = "RU" 
 	"0001041B" = "SL" 
 	"0001041F" = "TR" 
 	"00010426" = "US" 
 	"00010C0C" = "CF" 
 	"00010C1A" = "US" 
 	"00020408" = "GK" 
 	"00020409" = "US" 
 	"00030409" = "USL" 
 	"00040409" = "USR" 
 	"00050408" = "GK" 
 	}
 	$keyb = $keyboardmap.$($ObjKeyboards.Layout)
 	if (!$keyb)
 	{ $keyb = "Unknown"
 	}
 	Write-Output "..Event Log Settings"
 	$colLogFiles = Get-WmiObject -ComputerName $Target Win32_NTEventLogFile
 	Write-Output "..Event Log Errors"
 	$WmidtQueryDT = [System.Management.ManagementDateTimeConverter]::ToDmtfDateTime([DateTime]::Now.AddDays(-14))
 	$colLoggedEvents = Get-WmiObject -computer $Target -query ("Select * from Win32_NTLogEvent Where Type='Error' and TimeWritten >='" + $WmidtQueryDT + "'")
 	Write-Output "..Event Log Warnings"
 	$WmidtQueryDT = [System.Management.ManagementDateTimeConverter]::ToDmtfDateTime([DateTime]::Now.AddDays(-14))
 	$colLoggedEvents = Get-WmiObject -computer $Target -query ("Select * from Win32_NTLogEvent Where Type='Warning' and TimeWritten >='" + $WmidtQueryDT + "'")
 
 	$Filename = ".\" + $Target + "_" + $date.Hour + $date.Minute + "_" + $Date.Day + "-" + $Date.Month + "-" + $Date.Year + ".htm"
 
 	$Report = @"
 	<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN">
 	<html ES_auditInitialized='false'><head><title>Audit</title>
 	<META http-equiv=Content-Type content='text/html; charset=windows-1252'>
 	<STYLE type=text/css>	
 		TABLE {TABLE-LAYOUT: fixed; FONT-SIZE: 100%; WIDTH: 100%}
 		td {VERTICAL-ALIGN: TOP; FONT-FAMILY: Tahoma}
 	</STYLE>
 	<SCRIPT language=vbscript>
 		strShowHide = 1
 		strShow = "show"
 		strHide = "hide"
 		strShowAll = "show all"
 		strHideAll = "hide all"
 	
 	Function window_onload()
 		If UCase(document.documentElement.getAttribute("ES_auditInitialized")) <> "TRUE" Then
 			Set objBody = document.body.all
 			For Each obji in objBody
 				If IsSectionHeader(obji) Then
 					If IsSectionExpandedByDefault(obji) Then
 						ShowSection obji
 					Else
 						HideSection obji
 					End If
 				End If
 			Next
 			objshowhide.innerText = strShowAll
 			document.documentElement.setAttribute "ES_auditInitialized", "true"
 		End If
 	End Function
 	
 	Function IsSectionExpandedByDefault(objHeader)
 		IsSectionExpandedByDefault = (Right(objHeader.className, Len("_expanded")) = "_expanded")
 	End Function
 	
 	Function document_onclick()
 		Set strsrc = window.event.srcElement
 		While (strsrc.className = "sectionTitle" or strsrc.className = "expando")
 			Set strsrc = strsrc.parentElement
 		Wend
 		If Not IsSectionHeader(strsrc) Then Exit Function
 		ToggleSection strsrc
 		window.event.returnValue = False
 	End Function
 	
 	Sub ToggleSection(objHeader)
 		SetSectionState objHeader, "toggle"
 	End Sub
 	
 	Sub SetSectionState(objHeader, strState)
 		i = objHeader.sourceIndex
 		Set all = objHeader.parentElement.document.all
 		While (all(i).className <> "container")
 			i = i + 1
 		Wend
 		Set objContainer = all(i)
 		If strState = "toggle" Then
 			If objContainer.style.display = "none" Then
 				SetSectionState objHeader, "show" 
 			Else
 				SetSectionState objHeader, "hide" 
 			End If
 		Else
 			Set objExpando = objHeader.children.item(1)
 			If strState = "show" Then
 				objContainer.style.display = "block" 
 				objExpando.innerText = strHide
 	
 			ElseIf strState = "hide" Then
 				objContainer.style.display = "none" 
 				objExpando.innerText = strShow
 			End If
 		End If
 	End Sub
 	
 	Function objshowhide_onClick()
 		Set objBody = document.body.all
 		Select Case strShowHide
 			Case 0
 				strShowHide = 1
 				objshowhide.innerText = strShowAll
 				For Each obji In objBody
 					If IsSectionHeader(obji) Then
 						HideSection obji
 					End If
 				Next
 			Case 1
 				strShowHide = 0
 				objshowhide.innerText = strHideAll
 				For Each obji In objBody
 					If IsSectionHeader(obji) Then
 						ShowSection obji
 					End If
 				Next
 		End Select
 	End Function
 	
 	Function IsSectionHeader(obj) : IsSectionHeader = (obj.className = "heading0_expanded") Or (obj.className = "heading1_expanded") Or (obj.className = "heading1") Or (obj.className = "heading2"): End Function
 	Sub HideSection(objHeader) : SetSectionState objHeader, "hide" : End Sub
 	Sub ShowSection(objHeader) : SetSectionState objHeader, "show": End Sub
 	</SCRIPT>
 	</HEAD>
 	<BODY>
 	<font face="Arial" size="1"><b><i>Version 1.1 by Alan Renouf (Virtu-Al)</i></b></font><br>
 	<font face="Arial" size="1">Report generated on $Date </font></p>
 	
 	<TABLE cellSpacing=0 cellPadding=0>
 		<TBODY>
 			<TR>
 				<TD>
 					<DIV id=objshowhide tabIndex=0><FONT face=Arial></FONT></DIV>
 				</TD>
 			</TR>
 		</TBODY>
 	</TABLE>
 	<DIV class=heading0_expanded>
 		<SPAN class=sectionTitle tabIndex=0>$target Details</SPAN>
 		<A class=expando href='#'></A>
 	</DIV>
 	<DIV class=filler></DIV>
 	<DIV class=container>
 	<DIV class=heading1>
 	<SPAN class=sectionTitle tabIndex=0>General</SPAN>
 	<A class=expando href='#'></A>
 	</DIV>
 	<DIV class=container>
 	<DIV class=tableDetail>
 	<TABLE>
 	<tr>
 	<th width='25%'><b>Computer Name</b></font></th>
 	<td width='75%'>$($ComputerSystem.Name)</font></td>
 	</tr>
 	<tr>
 	<th width='25%'><b>Computer Role</b></font></th>
 	<td width='75%'> $ComputerRole </font></td>
 	</tr>
 	<tr>
 	<th width='25%'><b>$CompType</b></font></th>	
 	<td width='75%'>$($ComputerSystem.Domain)</font></td>
 	</tr>
 	<tr>
 	<th width='25%'><b>Operating System</b></font></th>
 	<td width='75%'>$($OperatingSystems.Caption)</font></td>
 	</tr>
 	<tr>
 	<th width='25%'><b>Service Pack</b></font></th>
 	<td width='75%'>$($OperatingSystems.CSDVersion)</font></td>
 	</tr>
 	<tr>
 		<th width='25%'><b>System Root</b></font></th>
 		<td width='75%'>$($OperatingSystems.SystemDrive)</font></td>
 	</tr>
 	<tr>
 		<th width='25%'><b>Manufacturer</b></font></th>
 		<td width='75%'>$($ComputerSystem.Manufacturer)</font></td>
 	 				</tr>
 	<tr>
 		<th width='25%'><b>Model</b></font></th>
 		<td width='75%'>$($ComputerSystem.Model)</font></td>
 	 				</tr>
 	<tr>
 		<th width='25%'><b>Number of Processors</b></font></th>
 		<td width='75%'>$($ComputerSystem.NumberOfProcessors)</font></td>
 	 				</tr>
 	<tr>
 		<th width='25%'><b>Memory</b></font></th>
 		<td width='75%'>$($ComputerSystem.TotalPhysicalMemory)</font></td>
 	 				</tr>
 	<tr>
 		<th width='25%'><b>Registered User</b></font></th>
 		<td width='75%'>$($ComputerSystem.PrimaryOwnerName)</font></td>
 	 				</tr>
 	 				<tr>
 		<th width='25%'><b>Registered Organisation</b></font></th>
 		<td width='75%'>$($OperatingSystems.Organization)</font></td>
 	</tr>
 	  				<tr>
 	   				<th width='25%'><b>Last System Boot</b></font></th>
 		<td width='75%'>$LBTime</font></td>
 	</tr>
 				</TABLE>
 			</DIV>
 		</DIV>
 		<DIV class=filler></DIV>
 	<DIV class=container>
 	<DIV class=heading1>
 	<SPAN class=sectionTitle tabIndex=0>HotFixes</SPAN>
 	<A class=expando href='#'></A>
 	</DIV>
 	<DIV class=container>
 	<DIV class=tableDetail>
 	<TABLE>
 	<tr>
 	<th width='25%'><b>HotFix Number</b></font></th>
 	<th width='75%'><b>Description</b></font></th>
 	</tr>
 "@
 
 	ForEach ($objQuickFix in $colQuickFixes)
 	{
 		if ($objQuickFix.HotFixID -ne "File 1")
 		{
 			$Report+= "				<tr>"
 			$Report+= "					<td width='25%'>$($objQuickFix.HotFixID)</font></td>"
 			$Report+= "					<td width='75%'>$($objQuickFix.Description)</font></td>"
 			$Report+= "				</tr>"
 		}
 	}
 $Report+= @"
 	</TABLE>
 	</DIV>
 	</DIV>
 	</DIV>
 	<DIV class=filler></DIV>
 		<DIV class=container>
 			<DIV class=heading1>
 				<SPAN class=sectionTitle tabIndex=0>Logical Disk Configuration</SPAN>
 				<A class=expando href='#'></A>
 			</DIV>
 			<DIV class=container>
 				<DIV class=tableDetail>
 					<TABLE>
 						<tr>
 	  						<th width='15%'><b>Drive Letter</b></font></th>
 	  						<th width='20%'><b>Label</b></font></th>
 	  						<th width='20%'><b>File System</b></font></th>
 	  						<th width='15%'><b>Disk Size</b></font></th>
 	  						<th width='15%'><b>Disk Free Space</b></font></th>
 	  						<th width='15%'><b>% Free Space</b></font></th>
 	  					</tr>
 "@
 
 	Foreach ($objDisk in $colDisks)
 	{
 		if ($objDisk.DriveType -eq 3)
 		{
 			$Report+=  "					<tr>"
 			$Report+=  "						<td width='15%'>$($objDisk.DeviceID)</font></td>"
 			$Report+=  " 						<td width='20%'>$($objDisk.VolumeName)</font></td>"
 			$Report+=  " 						<td width='20%'>$($objDisk.FileSystem)</font></td>"
 			$disksize = [math]::round(($objDisk.size / 1048576))
 			$Report+=  " 						<td width='15%'>$disksize MB</font></td>"
 			$freespace = [math]::round(($objDisk.FreeSpace / 1048576))
 			$Report+=  " 						<td width='15%'>$Freespace MB</font></td>"
 			$percFreespace=[math]::round(((($objDisk.FreeSpace / 1048576)/($objDisk.Size / 1048676)) * 100),0)
 			$Report+=  " 						<td width='15%'>$percFreespace%</font></td>"
 			$Report+=  "					</tr>"
 		}
 	}
 $Report+= @"
 	</TABLE>
 	</DIV>
 	</DIV>
 	</DIV>
 	<DIV class=filler></DIV>
 
 	
 		<DIV class=container>
 			<DIV class=heading1>
 				<SPAN class=sectionTitle tabIndex=0>NIC Configuration</SPAN>
 				<A class=expando href='#'></A>
 			</DIV>
 			<DIV class=container>
 				<DIV class=tableDetail>
 					<TABLE>
 "@
 	Foreach ($objAdapter in $colAdapters)
 	{
 		if ($objAdapter.IPEnabled -eq "True")
 		{
 			$NICCount = $NICCount + 1
 			If ($NICCount -gt 1)
 			{
 				$Report+=  "			</TABLE>"
 				$Report+=  "				<DIV class=Solidfiller></DIV>"
 				$Report+=  "			<TABLE>"
 			}
 		$Report+=  "  					<tr>"
 		$Report+=  "	 					<th width='25%'><b>Description</b></font></th>"
 		$Report+=  "    					<td width='75%'>$($objAdapter.Description)</font></td>"
 		$Report+=  "  					</tr>"
 		$Report+=  "  					<tr>"
 		$Report+=  "						<th width='25%'><b>Physical address</b></font></th>"
 		$Report+=  "						<td width='75%'>$($objAdapter.MACaddress)</font></td>"
 		$Report+=  " 					</tr>"
 		If ($objAdapter.IPAddress -ne $Null)
 		{
 			$Report+=  "					<tr>"
 			$Report+=  "						<th width='25%'><b>IP Address / Subnet Mask</b></font></th>"
 			$Report+=  "						<td width='75%'>$($objAdapter.IPAddress)/$($objAdapter.IPSubnet)</font></td>"
 			$Report+=  "					</tr>"
 			$Report+=  "					</tr>"
 			$Report+=  "					<tr>"
 			$Report+=  "						<th width='25%'><b>Default Gateway</b></font></th>"
 			$Report+=  "						<td width='75%'>$($objAdapter.DefaultIPGateway)</font></td>"
 			$Report+=  "					</tr>"
 		
 		}
 		$Report+=  "					<tr>"
 		$Report+=  "						<th width='25%'><b>DHCP enabled</b></font></th>"
 		If ($objAdapter.DHCPEnabled -eq "True")
 		{
 			$Report+=  "						<td width='75%'>Yes</font></td>"
 		}
 		Else
 		{
 			$Report+=  "						<td width='75%'>No</font></td>"
 		}
 		$Report+=  "					</tr>"
 		$Report+=  "					<tr>"
 		$Report+=  "							<th width='25%'><b>DNS Servers</b></font></th>"
 		$Report+=  "							<td width='75%'>"
 		If ($objAdapter.DNSServerSearchOrder -ne $Null)
 		{
 			$Report+=  " $($objAdapter.DNSServerSearchOrder) "
 		}
 		$Report+=  "					</tr>"
 		$Report+=  "					<tr>"
 		$Report+=  "						<th width='25%'><b>Primary WINS Server</b></font></th>"
 		$Report+=  "						<td width='75%'>$($objAdapter.WINSPrimaryServer)</font></td>"
 		$Report+=  "					</tr>"
 		$Report+=  "					<tr>"
 		$Report+=  "						<th width='25%'><b>Secondary WINS Server</b></font></th>"
 		$Report+=  "						<td width='75%'>$($objAdapter.WINSSecondaryServer)</font></td>"
 		$Report+=  "					</tr>"
 		$NICCount = $NICCount + 1
 		}
 	}
 $Report+= @"
 					</TABLE>
 				</DIV>
 			</DIV>
 		</DIV>
 		<DIV class=filler></DIV>
 "@	
 	if ((get-wmiobject -namespace "root/cimv2" -list) | ? {$_.name -match "Win32_Product"})
 	{
 		Write-Output "..Installed Software"
 		$colShares = get-wmiobject -ComputerName $Target Win32_Product | select Name,Version,Vendor,InstallDate
 		
 $Report+= @"
 	<DIV class=container>
 				<DIV class=heading1>
 					<SPAN class=sectionTitle tabIndex=0>Software</SPAN>
 					<A class=expando href='#'></A>
 				</DIV>
 				<DIV class=container>
 					<DIV class=tableDetail>
 						<TABLE>
 							<tr>
 		  						<th width='25%'><b>Name</b></font></th>
 		  						<th width='25%'><b>Version</b></font></th>
 		  						<th width='25%'><b>Vendor</b></font></th>
 		  						<th width='25%'><b>Install Date</b></font></th>
 							</tr>
 "@
 		Foreach ($objShare in $colShares)
 		{
 			$Report+=  "					<tr>"
 			$Report+=  "						<td width='50%'>$($objShare.Name)</font></td>"
 			$Report+=  "						<td width='20%'>$($objShare.Version)</font></td>"
 			$Report+=  "						<td width='15%'>$($objShare.Vendor)</font></td>"
 			$Report+=  "						<td width='15%'>$($objShare.InstallDate)</font></td>"
 			$Report+=  "					</tr>"
 		}
 		$Report+=  "				</TABLE>"
 		$Report+=  "			</DIV>"
 		$Report+=  "		</DIV>"
 		$Report+=  "	</DIV>"
 		$Report+=  "	<DIV class=filler></DIV>"		
 	}
 $Report+= @"
 		<DIV class=container>
 			<DIV class=heading1>
 				<SPAN class=sectionTitle tabIndex=0>Local Shares</SPAN>
 				<A class=expando href='#'></A>
 			</DIV>
 			<DIV class=container>
 				<DIV class=tableDetail>
 					<TABLE>
 						<tr>
 	  						<th width='25%'><b>Share</b></font></th>
 	  						<th width='25%'><b>Path</b></font></th>
 	  						<th width='50%'><b>Comment</b></font></th>
 						</tr>
 "@
 Foreach ($objShare in $colShares)
 	{
 		$Report+=  "					<tr>"
 		$Report+=  "						<td width='25%'>$($objShare.Name)</font></td>"
 		$Report+=  "						<td width='25%'>$($objShare.Path)</font></td>"
 		$Report+=  "						<td width='50%'>$($objShare.Caption)</font></td>"
 		$Report+=  "					</tr>"
 	}	
 $Report+= @"
 					</TABLE>
 				</DIV>
 			</DIV>
 		</DIV>
 		<DIV class=filler></DIV>
 	
 		<DIV class=container>
 			<DIV class=heading1>
 				<SPAN class=sectionTitle tabIndex=0>Printers</SPAN>
 				<A class=expando href='#'></A>
 			</DIV>
 			<DIV class=container>
 				<DIV class=tableDetail>
 					<TABLE>
 						<tr>
 							<th width='25%'><b>Printer</b></font></th>
 							<th width='25%'><b>Location</b></font></th>
 							<th width='25%'><b>Default Printer</b></font></th>
 							<th width='25%'><b>Portname</b></font></th>
 						</tr>
 "@
 	Foreach ($objPrinter in $colInstalledPrinters)
 	{
 		If ($objPrinter.Name -eq "")
 		{
 			$Report+=  "					<tr>"
 			$Report+=  "						<td width='100%'>No Printers Installed</font></td>"
 		}
 		Else
 		{
 			$Report+=  "					<tr>"
 			$Report+=  "						<td width='25%'>$($objPrinter.Name)</font></td>"
 			$Report+=  "						<td width='25%'>$($objPrinter.Location)</font></td>"
 			if ($objPrinter.Default -eq "True")
 			{
 				$Report+=  "						<td width='25%'>Yes</font></td>"
 			}
 			Else
 			{
 				$Report+=  "						<td width='25%'>No</font></td>"
 			}
 			$Report+=  "						<td width='25%'>$($objPrinter.Portname)</font></td>"
 		}
 		$Report+=  "					</tr>"
 	}
 $Report+= @"
 				</TABLE>
 				</DIV>
 			</DIV>
 		</DIV>
 		<DIV class=filler></DIV>
 		<DIV class=container>
 			<DIV class=heading1>
 				<SPAN class=sectionTitle tabIndex=0>Services</SPAN>
 				<A class=expando href='#'></A>
 			</DIV>
 			<DIV class=container>
 				<DIV class=tableDetail>
 					<TABLE>
 	  					<tr>
 		 					<th width='20%'><b>Name</b></font></th>
 		 					<th width='20%'><b>Account</b></font></th>
 		 					<th width='20%'><b>Start Mode</b></font></th>
 		 					<th width='20%'><b>State</b></font></th>
 		 					<th width='20%'><b>Expected State</b></font></th>
 	  					</tr>
 "@
 
 	Foreach ($objService in $colListOfServices)
 	{
 		$Report+=  " 					<tr>"
 		$Report+=  "	 					<td width='20%'>$($objService.Caption)</font></td>"
 		$Report+=  "	 					<td width='20%'>$($objService.Startname)</font></td>"
 		$Report+=  "	 					<td width='20%'>$($objService.StartMode)</font></td>"
 		If ($objService.StartMode -eq "Auto")
 		{
 			if ($objService.State -eq "Stopped")
 			{
 			}
 		}
 		If ($objService.StartMode -eq "Auto")
 		{
 			if ($objService.State -eq "Running")
 			{
 			}
 		}
 		If ($objService.StartMode -eq "Disabled")
 		{
 			If ($objService.State -eq "Running")
 			{
 			}
 		}
 		If ($objService.StartMode -eq "Disabled")
 		{
 			if ($objService.State -eq "Stopped")
 			{
 			}
 		}
 		If ($objService.StartMode -eq "Manual")
 		{
 		}
 		If ($objService.State -eq "Paused")
 		{
 		}
 		$Report+=  "  					</tr>"
 	}	
 $Report+= @"
 					</TABLE>
 				</DIV>
 			</DIV>
 		</DIV>
 		<DIV class=filler></DIV>
 		<DIV class=container>
 			<DIV class=heading1>
 				<SPAN class=sectionTitle tabIndex=0>Regional Settings</SPAN>
 				<A class=expando href='#'></A>
 			</DIV>
 			<DIV class=container>
 				<DIV class=tableDetail>
 					<TABLE>
 	 					<tr>
 		 					<th width='25%'><b>Time Zone</b></font></th>
 		 					<td width='75%'>$($TimeZone.Description)</font></td>
 	 					</tr>
 	 					<tr>
 		 					<th width='25%'><b>Country Code</b></font></th>
 		 					<td width='75%'>$($OperatingSystems.Countrycode)</font></td>
 	 					</tr>
 	 					<tr>
 			 				<th width='25%'><b>Locale</b></font></th>
 			 				<td width='75%'>$($OperatingSystems.Locale)</font></td>
 	 					</tr>
 	 					<tr>
 			 				<th width='25%'><b>Operating System Language</b></font></th>
 			 				<td width='75%'>$($OperatingSystems.OSLanguage)</font></td>
 	 					</tr>
 	 					<tr>
 		 				<th width='25%'><b>Keyboard Layout</b></font></th>
 			 				<td width='75%'>$keyb</font></td>
 	 					</tr>
 					</TABLE>
 				</div>
 			</DIV>
 		</DIV>
 		<DIV class=filler></DIV>
 		<DIV class=container>
 			<DIV class=heading1>
 				<SPAN class=sectionTitle tabIndex=0>Event Logs</SPAN>
 				<A class=expando href='#'></A>
 			</DIV>
 			<DIV class=container>
 				<DIV class=tableDetail>
 		<DIV class=container>
 			<DIV class=heading2>
 				<SPAN class=sectionTitle tabIndex=0>Event Log Settings</SPAN>
 				<A class=expando href='#'></A>
 			</DIV>
 			<DIV class=container>
 				<DIV class=tableDetail>
 					<TABLE>
 	  					<tr>
 	    					<th width='25%'><b>Log Name</b></font></th>
 	    					<th width='25%'><b>Overwrite Outdated Records</b></font></th>
 	  					  	<th width='25%'><b>Maximum Size</b></font></th>
 	 					   	<th width='25%'><b>Current Size</b></font></th>
 	 					</tr>
 "@
 	ForEach ($objLogFile in $colLogfiles)
 	{
 		$Report+=  " 					<tr>"
 		$Report+=  "	 					<td width='25%'>$($objLogFile.LogFileName)</font></td>"
 		If ($objLogfile.OverWriteOutdated -lt 0)
 		{
 			$Report+=  "	 					<td width='25%'>Never</font></td>"
 		}
 		if ($objLogFile.OverWriteOutdated -eq 0)
 		{
 			$Report+=  "	 					<td width='25%'>As needed</font></td>"
 		}
 		Else
 		{
 			$Report+=  "	 					<td width='25%'>After $($objLogFile.OverWriteOutdated) days</font></td>"
 		}
 		$MaxFileSize = ($objLogfile.MaxFileSize) / 1024
 		$FileSize = ($objLogfile.FileSize) / 1024
 		
 		$Report+=  "	 					<td width='25%'>$MaxFileSize KB</font></td>"
 		$Report+=  "	 					<td width='25%'>$FileSize KB</font></td>"
 		$Report+=  "  					</tr>"
 	}
 $Report+= @"
 					</TABLE>
 				</DIV>
 			</DIV>
 		</DIV>
 		<DIV class=filler></DIV>
 		<DIV class=container>
 			<DIV class=heading2>
 				<SPAN class=sectionTitle tabIndex=0>ERROR Entries</SPAN>
 				<A class=expando href='#'></A>
 			</DIV>
 			<DIV class=container>
 				<DIV class=tableDetail>
 					<TABLE>
 	  					<tr>
 	    					<th width='10%'><b>Event Code</b></font></th>
 	   					<th width='10%'><b>Source Name</b></font></th>
 	    					<th width='15%'><b>Time</b></font></th>
 	    					<th width='10%'><b>Log</b></font></th>
 	    					<th width='55%'><b>Message</b></font></th>
 	  					</tr>
 "@
 	ForEach ($objEvent in $colLoggedEvents)
 	{
 		$dtmEventDate = $ObjEvent.ConvertToDateTime($objEvent.TimeWritten)
 		$Report+=  " 					<tr>"
 		$Report+=  "	 					<td width='10%'>$($objEvent.EventCode)</font></td>"
 		$Report+=  "	 					<td width='10%'>$($objEvent.SourceName)</font></td>"
 		$Report+=  "	 					<td width='15%'>$dtmEventDate</font></td>"
 		$Report+=  "	 					<td width='10%'>$($objEvent.LogFile)</font></td>"
 		$Report+=  "	 					<td width='55%'>$($objEvent.Message)</font></td>"
 		$Report+=  "  					</tr>"
 	}
 $Report+= @"
 					</TABLE>
 				</DIV>
 			</DIV>
 		</DIV>
 		<DIV class=filler></DIV>
 		<DIV class=container>
 			<DIV class=heading2>
 				<SPAN class=sectionTitle tabIndex=0>WARNING Entries</SPAN>
 				<A class=expando href='#'></A>
 			</DIV>
 			<DIV class=container>
 				<DIV class=tableDetail>
 					<TABLE>
 	  					<tr>
 	    					<th width='10%'><b>Event Code</b></font></th>
 	   					<th width='10%'><b>Source Name</b></font></th>
 	    					<th width='15%'><b>Time</b></font></th>
 	    					<th width='10%'><b>Log</b></font></th>
 	    					<th width='55%'><b>Message</b></font></th>
 	  					</tr>
 "@
 	ForEach ($objEvent in $colLoggedEvents)
 	{
 		$StrWMIDate = $ObjEvent.ConvertToDateTime($objEvent.TimeWritten)
 		$Report+=  " 					<tr>"
 		$Report+=  "	 					<td width='10%'>$($objEvent.EventCode)</font></td>"
 		$Report+=  "	 					<td width='10%'>$($objEvent.SourceName)</font></td>"
 		$Report+=  "	 					<td width='15%'>$($dtmEventDate)</font></td>"
 		$Report+=  "	 					<td width='10%'>$($objEvent.LogFile)</font></td>"
 		$Report+=  "	 					<td width='55%'>$($objEvent.Message)</font></td>"
 		$Report+=  "  					</tr>"
 	}
 $Report+= @"
 					</TABLE>
 				</DIV>
 			</DIV>
 		</DIV>
 		<DIV class=filler></DIV>
 				</DIV>
 			</DIV>
 		</DIV>
 	
 		<DIV class=filler></DIV>
 	</body>
 	</html>
 "@
 $Report | out-file -encoding ASCII -filepath $Filename
 }
`

