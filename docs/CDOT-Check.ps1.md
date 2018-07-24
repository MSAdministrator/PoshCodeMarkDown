---
Author: vcosonok
Publisher: 
Copyright: 
Email: 
Version: 8.2
Encoding: ascii
License: cc0
PoshCode ID: 5498
Published Date: 2016-10-09t10
Archived Date: 2016-11-14t08
---

# cdot-check.ps1 - 

## Description

see www.cosonok.com and cdot-check.ps1

## Comments



## Usage



## TODO



## script

`pad-r`

## Code

`#
 #
 ##########################################################################
 ################################################################################
 ################################################################################
 
 $title = "CDOT-Check - V1 OCT '14 (For CDOT 8.2.X)"
 
 
 $GetNcCLUSTER    = ("Get-NcCluster","Get-NcSnmp","Get-NcOption","Get-NcTimezone","Get-NcNetDns","Get-NcNetDnsHost","Get-NcConfigBackupURL","Get-NcConfigBackupCount","Get-NcLicense",
                     "Get-NcEmsConfig","Get-NcEmsDestination","Get-NcJobCronSchedule","Get-NcJobIntervalSchedule","Get-NcKerberosRealm",
 					"Get-NcNetOption","Get-NcNetRoutingGroupRoute","Get-NcNetInterface",
 					"Get-NcSnapmirrorPolicy","Get-NcSecurityCertificate","Get-NcSecuritySSL","Get-NcSnapshotPolicy")
 $GetNcCLUSTERLR  = ("Get-NcUser","Get-NcRoleConfig","Get-NcRole")
 $GetNcNODE       = ("Get-NcNode -Name","Get-NcSystemImage -Node","Get-NcServiceProcessor -Node","Get-NcOption -Vserver","Get-NcAutoSupportConfig -Node","Get-NcClusterHA","Get-NcVol -Vserver",
                     "Get-NcNetRoutingGroupRoute","Get-NcNetInterface -Vserver",
 					"Get-NcNetPort -Node","Get-NcNetPortIfgrp -Node","Get-NcNetPortVlan -Node",
 					"Get-NcAggrNodeInfo -Name","Get-NcSecurityCertificate -Vserver","Get-NcSecuritySSL -Vserver")
 $GetNcAGGR       = ("Get-NcAggr","Get-NcAggrOption")
 $GetNcSVM        = ("Get-NcVserver","Get-NcOption","Get-NcNetDns","Get-NcNetDnsHost","Get-NcFileServiceAudit","Get-NcFlexcachePolicy",
                     "Get-NcFpolicyStatus","Get-NcFpolicyPolicy","Get-NcFpolicyEvent",
 					"Get-NcKerberosConfig","Get-NcLdapConfig","Get-NcLdapClient",
 					"Get-NcNetRoutingGroupRoute","Get-NcNetInterface",
 					"Get-NcSnapmirrorPolicy","Get-NcSecurityCertificate","Get-NcSecuritySSL","Get-NcSisPolicy","Get-NcSnapshotPolicy")
 $GetNcSVMLR      = ("Get-NcUser","Get-NcRoleConfig","Get-NcRole")
 $GetNcSVMCIFS    = ("Get-NcCifsServer","Get-NcCifsPreferredDomainController","Get-NcGpo","Get-NcCifsOption","Get-NcCifsSecurity","Get-NcCifsHomeDirectorySearchPath",
 	                "Get-NcCifsLocalUser","Get-NcCifsLocalGroup","Get-NcCifsLocalGroupMember","Get-NcCifsShareAcl","Get-NcCifsShare")
 $GetNcSVMNFS     = ("Get-NcNfsService","Get-NcNis",
                     "Get-NcNameMapping","Get-NcNameMappingUnixGroup","Get-NcNameMappingUnixUser",
 					"Get-NcExportPolicy","Get-NcExportRule","Get-NcNfsExport")
 $GetNcSVMVOLUMES = ("Get-NcQtree","Get-NcQuota","Get-NcQuotaStatus",
 					"Get-NcSis","Get-NcSnapshotAutodelete","Get-NcSnapshotReserve",
 					"Get-NcVol","Get-NcVolAutosize","Get-NcVolLanguage","Get-NcVolOption")
 
 $clusterConfigurations = $GetNcCLUSTER + $GetNcCLUSTERLR
 $nodeConfigurations    = $GetNcNODE + $GetNcAGGR
 $SVMConfigurations     = $GetNcSVM + $GetNcSVMLR + $GetNcSVMCIFS + $GetNcSVMNFS
 
 ################################################################################
 ################################################################################
 
 Function Pad-R{ Param([string]$string, [int]$int=30)
 	$string.PadRight($int," ").SubString(0,$int) }
 Function Pad-L{ Param([string]$in, [int]$padL=3, [string]$pad=" ")
 	$in.PadLeft($padL,$pad) }
 
 Function Columnize-W{
 	$i = 0
 	while($i -lt $args.count){
         Write-Host ((Pad-R $args[$i] $args[$i+1]) + " ") -F White -NoNewline
         $i += 2 }; Write-Host}
 
 Function Columnize-C{
 	$i = 0
 	while($i -lt $args.count){
         Write-Host ((Pad-R $args[$i] $args[$i+1]) + " ") -F $args[$i+2] -NoNewline
         $i += 3 }; Write-Host}
 
 Function Wr-E{Write-Host}
 Function Wr-C{If($args[0]){$P=$args[0]};Write-Host $P -F Cyan}
 Function Wr-D{If($args[0]){$P=$args[0]};Write-Host $P -F DarkGray}
 Function Wr-G{If($args[0]){$P=$args[0]};Write-Host $P -F Green}
 Function Wr-M{If($args[0]){$P=$args[0]};Write-Host $P -F Magenta}
 Function Wr-R{If($args[0]){$P=$args[0]};Write-Host $P -F Red}
 Function Wr-W{If($args[0]){$P=$args[0]};Write-Host $P -F White}
 Function Wr-Y{If($args[0]){$P=$args[0]};Write-Host $P -F Yellow}
 Function Wn-C{If($args[0]){$P=$args[0]};Write-Host $P -F Cyan -NoNewline}
 Function Wn-D{If($args[0]){$P=$args[0]};Write-Host $P -F DarkGray -NoNewline}
 Function Wn-G{If($args[0]){$P=$args[0]};Write-Host $P -F Green -NoNewline}
 Function Wn-R{If($args[0]){$P=$args[0]};Write-Host $P -F Red -NoNewline}
 Function Wn-W{If($args[0]){$P=$args[0]};Write-Host $P -F White -NoNewline}
 Function Wn-Y{If($args[0]){$P=$args[0]};Write-Host $P -F Yellow -NoNewline}
 Function Rd-W{If($args){Write-Host ($args[0]) -F White -NoNewline}; return (Read-Host "?")}
 
 Function Prompt-Keys{
 	Param([switch]$anykey)
 	If(!$anykey -and !$args[0]){ return $null }
 	If($args[0]){
 		If($args[0].count -ne 1){ $answers = $args[0] }
 		else { $answers  = $args[0..(($args.count)-1)] } }
 	while($true){
 		$keyPress = $host.UI.RawUI.ReadKey("NoEcho,IncludeKeyDown")
 		$keyPress = $keyPress.character.ToString().ToUpper()
 		If($anyKey){return $keypress}
 		foreach($answer in $answers){
 			if($keyPress -eq $answer.ToUpper()){ return $keyPress } }
 		Write-Host " " -NoNewLine; Write-Host "`b" -NoNewLine } }
 
 Function Prompt-Menu{
 	$question = $args[0]
 	If(($args[1].count) -ne 1){$answers = $args[1]}
 	else{$answers  = $args[1..(($args.count)-1)]}
 	while($true){
 		$readIn = Read-Host "$question"
 		$readIn = $readIn.ToUpper()
 		foreach($answer in $answers){
 			if($readIn -eq $answer.ToUpper()){return $readIn} }	} }
 
 Function Set-Window{
 	Param([string]$textcolor,[string]$background,[string]$title,[int]$percentMax)
 	$window = (get-host).UI.RawUI
 	If($textcolor)      {$window.ForegroundColor = $textcolor}
 	If($background -and ($window.BackgroundColor -ne $background)){
         $window.BackgroundColor = $background;cls}
 	If($title)          {$window.WindowTitle = $title}
 	$buffer            = $window.BufferSize
 	$buffer.Height     = 9999
 	$window.BufferSize = $buffer
 	If($perCentMax){
 		$maxX = [int](($window.MaxPhysicalWindowSize.Width)*$percentMax/100)
 		$maxY = [int](($window.MaxPhysicalWindowSize.Height)*$percentMax/100)
 		$buffer.Width = $maxX
 		$window.BufferSize = $buffer
 		$size = $window.WindowSize
 		$size.Width = $maxX
 		$size.Height = $maxY
 		$window.WindowSize = $size } }
 
 Function Format-Units{
 	Param([int64]$raw,[string]$unit)
 	$unit = $unit.ToUpper()
 	If($unit -eq "GB"){return (([int64]($raw/(1024*1024*1024))).ToString() + " $unit")} }
 
 Function Create-Folder {
 	If(!$args[0]){ return $null }
 	If(Test-Path $args[0] -ErrorAction SilentlyContinue){ return ($args[0]) }
 	Else {[Void](New-item $args[0] -type directory -ErrorAction SilentlyContinue)}
 	If(Test-Path $args[0] -ErrorAction SilentlyContinue){ return ($args[0]) }
 	Else { return $null } }
 
 ################################################################################
 ################################################################################
 
 Function Check-PSTK{
 	if(!(Get-Module DataONTAP)){Import-Module DataONTAP -ErrorAction SilentlyContinue}
 	if(!(Get-Module DataONTAP)){return $null}
 	$true}
 	
 Function Check-PSTKversion{
 	Param($major,$minor)
 	$ver = Get-NaToolKitVersion
 	if($ver.major -lt $major){return $null}elseif($ver.major -gt $major){return $true}
 	if($ver.minor -lt $minor){return $null}
 	$true}
 	
 Function Current-Cluster {
 	If($global:currentnccontroller.name){ $ClusterName = (Get-NcCluster).ClusterName } else { return $null}
 	If($ClusterName){ return $ClusterName } else { Current-ClusterNull; return $null } }
 Function Current-ClusterIP  {$global:currentnccontroller.Address.IPAddressToString}
 Function Current-ClusterNull{$global:currentnccontroller = $null}
 Function Current-UserName   {$global:currentnccontroller.Credentials.Username}
 Function Current-Vserver    {$global:currentnccontroller.Vserver}
 Function Current-VserverSet {$global:currentnccontroller.Vserver = $args[0]}
 Function Current-VserverNull{$global:currentnccontroller.Vserver = $null}
 
 ################################################################################
 ################################################################################
 
 Function Cluster-Connect{
 	Wr-W ">>>>>>> Connect to Cluster"; Wr-E
 	while ($true) {
 		Wr-W "Enter Netbios/FQDN/IP of Cluster to connect to (or enter to return to MAIN MENU)?";Wr-E
 		$cluster = Read-Host; Wr-E
 		If(!$cluster){return}
 		$getCreds = Get-Creds; Wr-E
 		If($getCreds){return} } }
 
 Function Get-Creds{
 	$filePath = $workingPath + "\" + $cluster + "-" + $whoAmI + ".cred"
 	Wn-W "Checking for credentials at ";Wr-C $filepath
 	$check = Test-Path $filePath
 	If (!$check){
 		Wr-Y "No saved credentials for $cluster detected!"; Wr-E
 		$PromptForCreds = Prompt-ForCreds
 		If($PromptForCreds){return $true} else {return $null}
 	} else {
 		$readFile = Get-Content $filePath
 		Wr-G "Saved credentials for $cluster detected with user $rdUsername!"; Wr-E
 		Wn-G "<<<<< Re-use credentials <Y/N>: "
 		$key = Prompt-Keys "Y" "N"; Wr-Y $key; Wr-E
 		If($key -eq "Y"){
 			$UseSavedCreds  = Use-SavedCreds
 			If($UseSavedCreds) {return $true} else {return $null}}
 		If($key -eq "N"){
 			$PromptForCreds = Prompt-ForCreds
 			If($PromptForCreds){return $true} else {return $null}} } }
 
 Function Prompt-ForCreds{
 	Wr-W "Enter username to connect to $cluster (or enter to return)?"; Wr-E
 	$username = Read-Host; Wr-E
 	If(!$username){return $null}
 	Wr-W "Enter the password (or enter to return)?"; Wr-E
 	$securePassword = Read-Host -AsSecureString; Wr-E
 	If($securePassword.length -eq 0){return $null}
 	$password       = $securePassword | ConvertFrom-SecureString
 	$credential     = New-Object System.Management.Automation.PsCredential($username,$securePassword)
 	$connect        = Connect-NcController $cluster -Credential $credential -ErrorAction SilentlyContinue
 	If($connect){
 		Wn-G "Connected!"; Wn-W " Saved credentials to "; Wr-C $filePath
 		[Void](New-Item $filePath -Type file -Force)
 		$username,$password | Set-Content $filePath
 		return $true
 	} else {
 		Wr-R "Failed to connect!"
 		return $null } }
 
 Function Use-SavedCreds{
 	$securePassword = $rdPassword | ConvertTo-SecureString
 	$credential = New-Object System.Management.Automation.PsCredential($rdUsername,$securePassword)
 	$connect = Connect-NcController $cluster -Credential $credential -ErrorAction SilentlyContinue
 	If($connect){Wr-G "Connected!";return $true} else {Wr-R "Failed to connect!";return $false}}
 
 ################################################################################
 ################################################################################
 
 Function Check-Health{
 	
 	$SVMquery             = Get-NcVserver -Template
 	$SVMquery.VserverType = "data"
 	
 	Wr-W ">>>>> Get-NcDiagnosisSubsystem Checks"; Wr-E
 	Display-HC (get-ncdiagnosissubsystem cifs_ndo) $cCluster "health" "ok"
 	Display-HC (get-ncdiagnosissubsystem sas_connect) $cCluster "health" "ok"
 	Display-HC (get-ncdiagnosissubsystem switch_health) $cCluster "health" "ok"; Wr-E
 	
 	Wr-W ">>>>> Get-NcNode Checks"; Wr-E	
 	
 	Wr-W ">>>>> Get-NcAutoSupportConfig Checks"; Wr-E
 	$nodes | foreach {Check-ASUP ($_.node)}; Wr-E
 
 	Wr-W ">>>>> Get-NcAutoSupportHistory Checks"; Wr-E
 	$nodes | foreach {Check-AsupSent ($_.node)}; Wr-E
 	
 	Wr-W ">>>>> Get-NcNetPort Checks"; Wr-E
 	$nodes | foreach {Check-Ports ($_.node)}
 	
 	Wr-W ">>>>> Get-NcAggr Checks"; Wr-E
 	$nodes | foreach {Check-Aggrs $_}
 
 	Wr-W ">>>>> Get-NcEmsMessage Checks"; Wr-E
 		
 	Wr-W ">>>>> Get-NcVserver Checks"; Wr-E
 	$dataSVMs | foreach {Check-DataSVMs $_}; Wr-E
 
 	Wr-W ">>>>> Get-NcCifs Checks"; Wr-E
 	$dataSVMs | foreach {If(($_.AllowedProtocols).Contains("cifs")){$cifsServers++;Check-CIFS ($_.Vserver)}}
 	If($countCifsServers -eq 0){Wr-C "No CIFS enabled SVMs!"}; Wr-E
 		
 	Wr-W ">>>>> Get-NcNfsService Checks"; Wr-E
 	$dataSVMs | foreach {If(($_.AllowedProtocols).Contains("nfs")){$nfsServers++;Check-NFS ($_.Vserver)}}
 	If($nfsServers -eq 0){Wr-C "No NFS enabled SVMs!"}; Wr-E
 	
 }
 
 
 	Param(
 		$thingToDot,$toDisplay,$member,$greenResult,$memberOverride,$resultOverride,
 		[switch]$lt,[switch]$gt,[switch]$ni,[switch]$ct,[switch]$pass,[switch]$failY)
 	
 	$result = $thingToDot.$member
 	If($resultOverride){$outResult = " = " + $resultOverride} else {$outResult = " = " + $result}
 	If($memberOverride){$outString = (Pad-R ($toDisplay + "." + $memberOverride) 40) + $outResult}
 	else               {$outString = (Pad-R ($toDisplay + "." + $member)         40) + $outResult}
 
 
 }
 
 Function Check-Nodes {
 	Param($node,[int]$hoursUp = 24)
 	$nodeName = $node.Node
 	$secondsUptime = $hoursUp*60*60
 	Display-HC $node $nodeName "IsNodeHealthy" "true"
 	Display-HC $node $nodeName "NodeUptime" $secondsUptime -gt
 	Display-HC $node $nodeName "EnvFailedFanCount" "0"
 	Display-HC $node $nodeName "EnvFailedPowerSupplyCount" "0"
 	Display-HC $node $nodeName "EnvOverTemperature" -ni
 	Display-HC $node $nodeName "NvramBatteryStatus" "battery_ok"
 	Display-HC $node $nodeName "IsEpsilonNode"
 	Display-HC $node $nodeName "IsNodeClusterEligible" "true"
 	Wr-E}
 
 function Check-ASUP {
 	Param([string]$nodeName)
 	$attributes                  = Get-NcAutoSupportConfig -Template
 	$attributes.IsEnabled        = ""
 	$attributes.IsSupportEnabled = ""
 	$getAsupConf                 = Get-NcAutoSupportConfig -Node $nodeName -Attributes $attributes
 	Display-HC $getAsupConf $nodeName "IsEnabled" "true"
 	Display-HC $getAsupConf $nodeName "IsSupportEnabled" "true"}
 
 function Check-AsupSent {
 
 	Param([string]$nodeName)
 	
 
 	
 	$i = 0
 		if ($getAsupHistory[$i].status -eq "transmission_failed"){Wr-R "$outString Failed!";          return}
 		if ($getAsupHistory[$i].status -eq "sent-successful"){    Wr-R "$outString Sent Successful!"; return}
 		$i++}
 	
 	Wr-R "$outString Failed!"
 	
 }	
 	
 Function Check-Ports{
 	Param([string]$nodeName)
 	$ncNetPort = Get-NcNetPort -Node $nodeName
 	$ncNetPort | foreach {
 		$port     = $_.Port
 		$adminUp  = $_.IsAdministrativeUp
 		$link     = $_.LinkStatus
 		$nodePort = "$nodeName" + ":" + "$port"
 		$P = "$nodePort.LinkStatus = $link (and IsAdminUp = $adminUp)";If($adminUp -and ($link -eq "up")){Wr-G}else{Wr-R}
 		Display-HC $_ $port "OperationalDuplex" "full"
 		Display-HC $_ $port "AdministrativeDuplex" "full"
 		Display-HC $_ $port "OperationalFlowcontrol" "none"
 		Display-HC $_ $port "AdministrativeFlowcontrol" "none"
 		Display-HC $_ $port "OperationalSpeed"
 		Display-HC $_ $port "AdministrativeSpeed" "auto"
 		Display-HC $_ $port "IsOperationalAutoNegotiate" "True"
 		Display-HC $_ $port "IsAdministrativeAutoNegotiate" "True"
 		Display-HC $_ $port "Mtu"
 		Wr-E}}
 
 Function Check-Aggrs{
 
 	Param($node)
 	
 	$volCount             = 0
 	$nodename             = $node.Node
 	$maxAggrSize          = $node.MaximumAggregateSize
 	$maxAggrSizeDisplay   = Format-Units $maxAggrSize GB
 	$maxAggrSizeThreshold = $maxAggrSize*0.95
 	
 	$NcAggrQuery       = Get-NcAggr -Template
 	$NcAggrQuery.Nodes = $nodename
 	$aggrs             = Get-NcAggr  -Query $NcAggrQuery
 
 	$aggrs | foreach {
 			
 		$root     = $false
 		$aggrname = $_.name
 
 		Display-HC $_ $aggrname "state" "online"
 		Display-HC $_ $aggrname "RaidType" "normal" -ct
 		Display-HC ($_.AggrOwnershipAttributes) $aggrname "HomeName" ($_.AggrOwnershipAttributes.OwnerName)
 
 		If($_.AggrRaidAttributes.HasLocalRoot){Display-HC ($_.AggrRaidAttributes) $aggrname "HasLocalRoot";$root = $true}
 		$modulo = ($_.Disks) % ($_.RaidSize)
 		If(!$root -and ($modulo -eq 0)){			
 				Display-HC $_ $aggrname "Disks" -Pass
 				Display-HC $_ $aggrname "RaidSize" -Pass
 		} elseif (!$root -and ($modulo -ne 0)) {
 				Display-HC $_ $aggrname "Disks" -FailY
 				Display-HC $_ $aggrname "RaidSize" -FailY
 		}
 
 		Display-HC $_ $aggrname "used" $aggrUsedThreshold "UsedPercent" -lt
 		Display-HC $_ $aggrname "TotalSize" $maxAggrSizeThreshold "TotalSize (max = $maxAggrSizeDisplay)" -r (Format-Units $_.TotalSize GB) -lt
 		Display-HC ($_.AggrSnapshotAttributes) $aggrname "SizeUsed" "0" "SnapshotSpaceUsed"
 		Display-HC ($_.AggrSnapshotAttributes) $aggrname "SnapshotReservePercent" "0"
 		Display-HC ($_.AggrPerformanceAttributes) $aggrname "FreeSpaceRealloc"
 
 		Wr-E
 		$volCount += $_.volumes
 		
 	
 	$maxVols  = $_.MaximumNumberOfVolumes
 	$maxVolTH = $maxVols*0.95
 	$P = "$nodename has $volCount volumes from a maximum of $maxVols!";If ($volCount -lt $maxVolTH){Wr-G} else {Wr-R}
 	Wr-E
 
 } 
 
 Function Check-EventLog{
 
 	Param([string]$nodeName,[int]$hours = 24)
 	$attributes       = Get-NcEmsMessage -Template
 	$attributes.Event = ""
 	$attributes.Node  = ""
 	$attributes.Time  = ""
 	$severityList     = "EMERGENCY","R","ALERT","R","CRITICAL","R","ERROR","Y","WARNING","Y"
 	$sevListCount     = $severityList.count
 	
 	$i = 0 
 	while ($i -lt $sevListCount) {
 		$severity = $severityList[$i]
 		$colour   = $severityList[$i+1]
 		$ems      = Get-NcEmsMessage -Severity $severity -StartTime (Get-Date).AddHours(-$hours) -Attributes $attributes -Node $nodeName
 		$emsCount = $ems.count
 		$P = "$nodeName has $emsCount events for type $severity in the last $hours hours!"; If($emsCount -eq 0){Wr-G} elseif($colour -eq "R"){Wr-R} else{Wr-Y}
 		$uniqueMessages[$i/2] = Check-EventLogUniqueMessages
 		$i+=2
 	} 
 	Wr-E
 	
 	$i = 0
 	while ($i -lt $sevListCount) {
 		$uniqueMessageTable  = $uniqueMessages[$i/2]
 		$uniqueMessageCount  = $uniqueMessageTable.count
 		If ($uniqueMessageCount -ne 0){
 			$colour              = $severityList[$i+1]
 			Wr-W ("$nodeName occurrences of unique " + $severityList[$i] + " events in the last $hours hours >")
 			Columnize-W "Count" 5 "Last Occurrence" 20 "Event" 80
 			$y = 0
 			while ($y -lt $uniqueMessageCount) {
 				Columnize-C $uniqueMessageTable[3,$y] 5 $colour $uniqueMessageTable[0,$y] 20 $colour $uniqueMessageTable[2,$y] 80 $colour
 				$y++
 				If(!$uniqueMessageTable[0,$y]){$y = $uniqueMessageCount}
 			}
 			Wr-E
 		}
 		$i+=2
 	}
 }
 
 Function Check-EventLogUniqueMessages {
 	$outputArray = New-Object 'object[,]' 4,$emsCount
 	$arrayYsize = 0 
 	foreach ($message in $ems){
 		$match = Check-EventLogMessageMatch
 		if (!$match){
 			$outputArray[0,$arrayYsize]=$message.TimeDT
 			$outputArray[1,$arrayYsize]=$message.Node
 			$outputArray[2,$arrayYsize]=$message.Event
 			$outputArray[3,$arrayYsize]=1
 			$arrayYsize++	
 		}
 	, $outputArray
 }
 
 Function Check-EventLogMessageMatch{
 	$j=0
 		if ($message.Event -eq $outputArray[2,$j]){
 		}
 		$j++
 	}
 	return $false
 } 
 
 Function Check-DataSVMs {
 	Param($dataSVM)
 	Display-HC $dataSVM ($dataSVM.Vserver) "State" "running"}
 
 Function Check-CIFS {
 	Param([string]$currentVserver)
 	Current-VserverSet $currentVserver
 	$getCifsServer = Get-NcCifsServer
 	If (!$getCifsServer){Wr-R "$currentVserver has CIFS allowed but no CIFS server created!"; Current-VserverNull; return}
 	$cifsServer    = $getCifsServer.CifsServer
 	Display-HC $getCifsServer $cifsServer "AdministrativeStatus" "up"
 	Current-VserverNull}
 
 Function Check-NFS{
 	Param([string]$currentVserver)
 	Current-VserverSet $currentVserver
 	$getNfsService = Get-NcNfsService
 	If (!$getNfsService){Wr-R "$currentVserver has NFS allowed but no NFS server created!"; Current-VserverNull; return}
 	Display-HC $getNfsService $currentVserver "GeneralAccess" "True"
 	Current-VserverNull}
 
 ################################################################################
 ###################################################################################
 ###################################################################################
 
 Function Get-Configuration {
 
 	
 	[string]$listOfNodes = "";$nodes | foreach {$listOfNodes += $_ + ","}
 	[string]$listOfSVMs  = "";$dataSVMs | foreach {$listOfSVMs += $_ + ","}
 	
 	
 	foreach ($getNc in $clusterConfigurations){
 			Get-CFGwDisplayAll CLUSTER $cCluster $getNc } 
 	
 	
 	foreach ($getNc in $GetNcNODE){
 		foreach ($node in $nodes){
 			Get-CFGwDisplayAll NODE $node $getNc } }
 
 	
 	$attributes = Get-NcAggr -template
 	$query      = Get-NcAggr -template
 	Initialize-NcObjectProperty -object $query -name AggrOwnershipAttributes	
 	foreach ($getNc in $GetNcAGGR){
 		foreach ($node in $nodes){
 			Wn-W ">>>>> NODE:          "; Wr-C $node
 			Wn-W ">>>>> Configuration: "; Wr-C $getNc; Wr-E
 			$query.AggrOwnershipAttributes.OwnerName = $node
 			$aggrs = (Get-NcAggr -Attributes $attributes -Query $query).Name | sort
 			$lastAggr = $aggrs[$aggrs.count-1]
 			foreach ($aggr in $aggrs){
 				$aggrHeader = "AGGREGATE".PadRight(30," ") + " = " + $aggr
 				Display-CI $aggrHeader
 				$cmdToDisplayAll = $getNc + " -Name " + $aggr
 				$output = Display-All $cmdToDisplayAll -All
 				$output | foreach {Display-CI $_}
 				If($aggr -ne $lastAggr) { Display-CI " " }
 			}			
 			Wr-E
 		}
 	}
 	
 	
 	foreach ($getNc in $SVMConfigurations){
 		foreach ($dataSVM in $dataSVMs){
 			Get-CFGwDisplayAll SVM $dataSVM $getNc } }
 			
 	
 	$attributes = Get-NcVol -template
 	foreach ($getNc in $GetNcSVMVOLUMES){
 		foreach ($dataSVM in $dataSVMs){
 			Current-VserverSet $dataSVM
 			Wn-W ">>>>> SVM:           "; Wr-C $dataSVM
 			Wn-W ">>>>> Configuration: "; Wr-C $getNc; Wr-E
 			$volumes = (Get-NcVol -Attributes $attributes).Name | sort
 			$lastVol = $volumes[$volumes.count-1]
 			If (($getNc -eq "Get-NcVolOption") -or ($getNc -eq "Get-NcVolLanguage") -or ($getNc -eq "Get-NcVolAutoSize"))
 				{ $switchParam = " -Name " } else { $switchParam = " -Volume " }
 			foreach ($volume in $volumes) {
 				$volHeader = "VOLUME".PadRight(30," ") + " = " + $volume
 				Display-CI $volHeader
 				$cmdToDisplayAll = $getNc + $switchParam + $volume
 				$output = Display-All $cmdToDisplayAll -All
 				$output | foreach {Display-CI $_}
 				If($volume -ne $lastVol) { Display-CI " " }
 			}
 			Wr-E		
 		}
 	}; Current-VserverNull
 
 }
 
 Function Display-CI { Param($member); $saveConfig += $member; Wr-C $member }
 
 	Param($EntityType,$SVMname,$GetNcCommand)	
 	$output = Display-All $cmdToDisplayAll -All
 	$EntityDisplay = ($EntityType + ":").PadRight(14)
 	Wn-W ">>>>> $EntityDisplay ";Wr-C $SVMname
 	Wn-W ">>>>> Configuration: ";Wr-C $cmdToDisplayAll; Wr-E
 	$output | foreach {Display-CI $_}
 	Current-VserverNull; Wr-E } 
 
 Function Save-Configuration{
 	Wr-W ">>>>>>> Save Configuration";Wr-E
 	$dateString = Get-Date -uformat "%Y%m%d"
 	Wn-W "Current folder for clustercfg files = "; Wr-C $clusterCfgFolder
 	Wn-W "Default filename                    = "; Wr-C "$cCluster.$dateString.clustercfg"; Wr-E
 	Wn-G "<<<<< Use default filename <Y/N>: "
 	$key = Prompt-Keys "Y" "N"; Wr-Y $key; Wr-E	
 	If ($key -eq "Y"){$saveFile = "$clusterCfgFolder\$cCluster.$dateString.clustercfg"}
 	If ($key -eq "N"){
 		$filename = Read-Host "Enter filename (.clustercfg is added)?";Wr-E
 		If(!$filename){return}
 		$saveFile = "$clusterCfgFolder\$filename.clustercfg"
 	}
 	[Void](New-Item $saveFile -Type file -Force)
 	Get-Configuration
 	$saveConfig | Out-File $saveFile
 }
 
 ################################################################################
 ################################################################################
 	
 Function Display-All {
 	Param($getCmd,[int]$padR = 30,[switch]$all = $null)
 	If(!$getCmd)   { return }
 	If($getCmd.gettype().name -eq "String"){$parsedCmd = Invoke-Expression $getCmd}
 	else {$parsedCmd = $getCmd}
 	If(!$parsedCmd){ return }
 	$parsedCmdCount = $parsedCmd.count
 
 Function Display-AllNameValuePairs{
 	$parsedCmd | foreach { ($_.Name).PadRight($padR," ").SubString(0,$padR) + " = " + ($_.Value) } }
 
 Function Display-AllMembers{
 	Param($element,[int]$padL=1)
 	($element|gm) | foreach{Member-Data $element $_} }
 	
 Function Member-Data {
 	Param($element,$member)
 	If($member.MemberType -eq "Method"){return}
 	$name    = $member.Name
 	$checks  = "Specified","NcController","Vserver","VserverName","VserverType","PublicCertificate"
 	foreach($check in $checks){If($name.Contains($check)){return}}
 	$data    = $element.$name
 	$padding = $name.length + $padL
 	$output  = $name.PadLeft($padding,".").PadRight($padR," ").SubString(0,$padR) + " = " + $data
 	If (!$data){If($all){return $output}else{return}}
 	$checks  = "bool","string[]","NcController","System.Object"
 	foreach($check in $checks){If($member.Definition.Contains($check)){return $output}}
 	$checks  = "String","Boolean","TimeSpan","DateTime","Int32","Int64"
 	foreach($check in $checks){If($data.gettype().name -eq $check){return $output}}
 	If (($data|gm).count -eq 0){return $output}
 	$name.PadLeft($padding,".");$padL++
 	$output  = Display-AllMembers $data $padL
 	return $output}
 
 ################################################################################
 ################################################################################
 
 Function Set-ClusterCFGFolder{
 	Wr-W "Enter folder/folderpath for clustercfg files?"; Wr-E
 	$readIn     = Read-Host; Wr-E
 	$testCreate = Create-Folder $readIn
 }
 
 Function Load-ClusterCfg {
 
 	Param([int]$fileNumber)
 	Wr-W ">>>>>>> Load ClusterCfg File"; Wr-E
 	Wn-W "Folder/folderpath for clustercfg files = "; Wr-C $clusterCfgFolder
 	$items        = Get-ChildItem -Path $clusterCfgFolder
 	$searchString = "*.clustercfg"
 	
 	If     ( $items.count -eq 0) { Wr-R "No *.clustercfg files in this folder!"; Wr-E; return $null} 
 	else   { $matches = $items -like $searchString }
 	If     (!$matches)           { Wr-R "No *.clustercfg files in this folder!"; Wr-E; return $null}
 	
 	Wr-E; Wr-W "Clustercfg Files in this Directory:"; Wr-E
 	
 	$i = 1
 	$promptOptions = @()
 	$promptOptions += "0"
 	Wr-W "  0) Unload clustercfg file and/or Return to Main Menu"
 	$matches | foreach{
 		$promptOptions += ($i.toString())
 		$display = Pad-L ($i.toString())
 		Wr-C "$display) $_"
 		$i++
 	}; Wr-E
 	
 	$answer = Prompt-Menu "Choose clustercfg file to load (or 0)?" $promptOptions
 	
 	$content        = Get-Content ($clusterCfgFolder + "\" + $chosenFilename) -ErrorAction SilentlyContinue
 	If(!$content){ Wr-E; Wr-R "Unable to load file!"; Wr-E; return}
 
 	$clusterCfg[$fileNumber,0] = $chosenFilename
 	$CfgContent[$fileNumber]   = $content
 	
 		
 	Wr-E
 	
 }
 
 Function Unload-ClusterCfg {
 
 	$clusterCfg[$fileNumber,0] = $clusterCfg[$fileNumber,1] = $clusterCfg[$fileNumber,2] = $null
 
 	If($CompareA.CLUSTERFILE -eq $fileNumber){$CompareA.CLUSTER = $null} 
 	If($CompareB.CLUSTERFILE -eq $fileNumber){$CompareB.CLUSTER = $null}
 	
 	If(($CompareA.NODEFILE -eq $fileNumber) -or ($cfgFiles -eq 1)){$CompareA.NODE = $null} 
 	If(($CompareB.NODEFILE -eq $fileNumber) -or ($cfgFiles -eq 1)){$CompareB.NODE = $null}
 	If(($CompareA.SVMFILE -eq $fileNumber ) -or ($cfgFiles -eq 1)){$CompareA.SVM = $null}
 	If(($CompareB.SVMFILE -eq $fileNumber ) -or ($cfgFiles -eq 1)){$CompareB.SVM = $null}
 
 }
 
 
 ################################################################################
 ################################################################################
 
 Function Select-PairToCompare{
 
 	Param([string]$EntityType)
 
 
 
 	Columnize-W "Menu Option" 20 "$EntityType Name" 20 "Cluster Name"       20 "Config File Number" 20
 	Columnize-C "0"           20 Yellow    "<Return to Main Menu>" 25 Cyan
 		Columnize-C $option 20 Yellow ($itemsMenu[$r,0]) 20 Green ($itemsMenu[$r,1]) 20 Cyan ($itemsMenu[$r,2].ToString()) 20 Cyan
 	
 	$answer0 = Prompt-Menu "Enter 1st item (to be compared with)    ?" $menuOptions; If($answer0 -eq "0"){ Wr-E; return}; $option0 = ([int]$answer0) -1
 	$answer1 = Prompt-Menu "Enter 2nd item (to compare with the 1st)?" $menuOptions; If($answer1 -eq "0"){ Wr-E; return}; $option1 = ([int]$answer1) -1; Wr-E
 	
 	
 }
 
 Function Compare-Data{
 	
 		
 	
 	$cfgFile = New-Object 'object[]' 2; $rowCount = New-Object 'object[]' 2; $rowsInFile = New-Object 'object[]' 2; $output = New-Object 'object[]' 2
 	$entityName1       = $CompareA.$EntityType
 	$entityName2       = $CompareB.$EntityType
 	$searchString      = New-Object 'object[,]' 2,2
 
 	
 	
 	Wn-W ">>>>> $EntityType Configuration Compare: "; Wr-C $check; Wr-E
 	Columnize-W "PROPERTY" $x1 "$entityName1" $x2 "$entityName2" $x3 "PROPERTY" $x4
 	
 	
 	
 	while ($haveContent){
 	
 
 
 		
 		
 
 
 Function Compare-DataChooseGetNc{
 
 	Param($entity,$list)
 	
 	Wr-W "Type a COMPLETE command from the following list, or type:"; Wr-E
 	Wr-W "    ALL to cycle through the complete list"
 	Wr-W "    EXIT to return"; Wr-E
 	
 	$list        = $list | sort ; $listCount    = $list.count
 	$menuChoices = @()          ; $menuChoices += "ALL","EXIT"; $l = 0	
 	while ($l -lt $listCount){
 		If( ($l+0) -lt $listCount){ $col0 = ($list[$l].substring(6).split(" "))[0];   $menuChoices += $col0 } else { $col0 = " " }
 		If( ($l+1) -lt $listCount){ $col1 = ($list[$l+1].substring(6).split(" "))[0]; $menuChoices += $col1 } else { $col1 = " " }
 		If( ($l+2) -lt $listCount){ $col2 = ($list[$l+2].substring(6).split(" "))[0]; $menuChoices += $col2 } else { $col2 = " " }
 		Columnize-C $col0 35 Cyan $col1 35 Cyan $col2 35 Cyan
 		$l+=3 }; Wr-E
 	
 	$answer = Prompt-Menu "Type a Get-Nc* command" $menuChoices; Wr-E
 	If($answer -eq "EXIT") { return }
 	If($answer -eq "ALL"){
 		Foreach ($GetNcCompare in $list){
 			Compare-Data $entity $GetNcCompare
 			Wr-G ">>>> Press ANY key to continue or X to E(X)IT"; Wr-E
 			$key = Prompt-Keys -AnyKey; If($key -eq "X"){ return } }
 		return }
 
 	$GetNcCompare = "Get-Nc" + $answer
 	
 	If($entity -eq "NODE"){
 		$i = 0
 		while ($i -lt $listCount){
 			If ($list[$i] -match $GetNcCompare){ $GetNcCompare = $list[$i]; $i = $listCount }
 			$i++ } } 
 
 	Compare-Data $entity $GetNcCompare
 	
 }
 
 ################################################################################
 ################################################################################
 
 
 If($toolkit -and !$versionChk){Wr-Y "Recommend DataONTAP PowerShell Toolkit version $TKmajor.$TKminor or better - older version detected!"};Wr-E
 
 
 	Wr-M "<<<<<<<<< MAIN MENU >>>>>>>>>"; Wr-E
 	
 	$cCluster = Current-Cluster; $cClusterIP = Current-ClusterIP; $cUsername = Current-Username
 	
 	If ($clusterCfg[0,0] -and $clusterCfg[1,0]){$cfgFiles = 2} elseif ($clusterCfg[0,0] -or  $clusterCfg[1,0]){$cfgFiles = 1} else {$cfgFiles = 0}
 
 			$CompareA.NODE = ( ($CfgContent[0][1]).Split(",") )[0]; $CompareA.NODEFILE = 0
 			$CompareB.NODE = ( ($CfgContent[1][1]).Split(",") )[0]; $CompareB.NODEFILE = 1
 			If ($clusterCfg[0,0]){
 				$CompareA.NODE = ( ($CfgContent[0][1]).Split(",") )[0]; $CompareA.NODEFILE = 0
 				$CompareB.NODE = ( ($CfgContent[0][1]).Split(",") )[1]; $CompareB.NODEFILE = 0 }		
 			If ($clusterCfg[1,0]){
 				$CompareA.NODE = ( ($CfgContent[1][1]).Split(",") )[0]; $CompareA.NODEFILE = 1
 				$CompareB.NODE = ( ($CfgContent[1][1]).Split(",") )[1]; $CompareB.NODEFILE = 1 } } }
 
 			$CompareA.SVM = ( ($CfgContent[0][3]).Split(",") )[0]; $CompareA.SVMFILE = 0
 			$CompareB.SVM = ( ($CfgContent[1][3]).Split(",") )[0]; $CompareB.SVMFILE = 1
 			If ($clusterCfg[0,0]){
 				$CompareA.SVM = ( ($CfgContent[0][3]).Split(",") )[0]; $CompareA.SVMFILE = 0
 				$CompareB.SVM = ( ($CfgContent[0][3]).Split(",") )[1]; $CompareB.SVMFILE = 0 }		
 			If ($clusterCfg[1,0]){
 				$CompareA.SVM = ( ($CfgContent[1][3]).Split(",") )[0]; $CompareA.SVMFILE = 1
 				$CompareB.SVM = ( ($CfgContent[1][3]).Split(",") )[1]; $CompareB.SVMFILE = 1 } } }
 
 	$clusterA = $CompareA.CLUSTER; $clusterB = $CompareB.CLUSTER; If($clusterA  -and $clusterB){$clusterPair  = $true} else {$clusterPair  = $null}   
 	$nodeA    = $CompareA.NODE;    $nodeB    = $CompareB.NODE;    If($nodeA     -and $nodeB)   {$nodePair     = $true} else {$nodePair     = $null}
 	$svmA     = $CompareA.SVM;     $svmB     = $CompareB.SVM;     If($svmA      -and $svmB)    {$svmPair      = $true} else {$svmPair      = $null}
 				
 	$P = "    < 1 > Connect to Cluster - PS toolkit not loaded!"; If(!$toolkit){Wr-D}
 	$P = "    < 1 > Connect to Cluster - "; If($toolkit -and !$cCluster){Wn-W;Wr-Y "no current connection!";$PROMPTKEYS += "1"}
 	$P = "    < 1 > Connect to Cluster -";  If($toolkit -and  $cCluster){Wn-W "$P current = ";Wn-C $cCluster;Wn-W " (IP ";Wn-C $cClusterIP;Wn-W ", user ";Wn-C $cUsername;Wr-W ")";$PROMPTKEYS += "1"}
 	$P = "    < 2 > Health Check Cluster";  If($cCluster) {Wr-W;$PROMPTKEYS += "2"} else {Wr-D}; Wr-E
 	Wn-W "    < 3 > Change Folder for saved clustercfg files - current = "; Wr-C $clusterCfgFolder
 	$P = "    < 4 > Save Current Cluster's Configuration"; If($cCluster)        {Wr-W;$PROMPTKEYS += "4"} else {Wr-D}
 	$P = "    < A > Load clustercfg File A -";             If($clusterCfg[0,0]) {Wn-W "$P loaded = "; Wr-C $clusterCfg[0,0]} else {Wr-W "$P nothing loaded!"}
 	$P = "    < B > Load clustercfg File B -";             If($clusterCfg[1,0]) {Wn-W "$P loaded = "; Wr-C $clusterCfg[1,0]} else {Wr-W "$P nothing loaded!"}
 	$P = "    Loaded  clustercfg files contain: ";    If($cfgFiles -ne 0)    {Wr-E; Wn-W; Wr-C "$cfgFiles CLUSTER(s), $cfgNodeCount NODE(s), and $cfgSVMCount Data SVM(s)"}; Wr-E
 	$P = "    < C > Compare CLUSTER  Configurations"; If($clusterPair)       {Wn-W "$P for ";Wn-C $clusterA;Wn-W " and ";Wr-C $clusterB;$PROMPTKEYS += "C"} else {Wr-D}
 	$P = "    < M > Select  NODEs to Compare";        If($cfgNodeCount -gt 2){Wr-W; $PROMPTKEYS += "M"}
 	$P = "    < N > Compare NODE     Configurations"; If($nodePair)          {Wn-W "$P for ";Wn-C $nodeA;Wn-W " and ";Wr-C $nodeB;$PROMPTKEYS += "N"} else {Wr-D}
 	$P = "    < R > Select  SVMs  to Compare";        If($cfgSVMCount -gt 2) {Wr-W; $PROMPTKEYS += "R"}
 	$P = "    < S > Compare SVM      Configurations"; If($svmPair)           {Wn-W "$P for ";Wn-C $svmA; Wn-W " and ";Wr-C $svmB; $PROMPTKEYS += "S"} else {Wr-D}; Wr-E
 	Wr-W "    < X > Exit"; Wr-E
 	$P = "    <<<<< Last pressed key: $lastPressed"; If($lastPressed) {Wr-W}
 	Wn-G "    >>>>> Press a Key: "
 	
     $key = Prompt-Keys $PROMPTKEYS;Wr-Y $key; Wr-E
 	If($key -eq "1"){ Cluster-Connect }
 	If($key -eq "2"){ Check-Health }
 	If($key -eq "3"){ $clusterCfgFolder = Set-ClusterCFGFolder }
 	If($key -eq "4"){ Save-Configuration }
 	If($key -eq "A"){ Load-ClusterCfg 0 }
 	If($key -eq "B"){ Load-ClusterCfg 1 }
 	If($key -eq "C"){ Compare-DataChooseGetNc CLUSTER $clusterConfigurations }
 	If($key -eq "M"){ Select-PairToCompare NODE }
 	If($key -eq "N"){ Compare-DataChooseGetNc NODE    $nodeConfigurations }
 	If($key -eq "R"){ Select-PairToCompare SVM }
 	If($key -eq "S"){ Compare-DataChooseGetNc SVM     ($SVMConfigurations + $GetNcSVMVOLUMEs) }
 	If($key -eq "X"){ EXIT }
 	$lastPressed = $key
 	
`

