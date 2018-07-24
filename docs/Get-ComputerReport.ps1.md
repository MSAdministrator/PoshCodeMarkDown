---
Author: chris weislak
Publisher: 
Copyright: 
Email: 
Version: 1.6
Encoding: ascii
License: cc0
PoshCode ID: 5476
Published Date: 2015-10-01t16
Archived Date: 2015-05-03t19
---

# get-computerreport - 

## Description

get a full html computer report from a list of computers or just one. the outpath is the folder to store the report. it includes scheduled tasks, applications, services and more…

## Comments



## Usage



## TODO



## function

`get-computerreport`

## Code

`#
 #
 Function Get-ComputerReport {
 	[CmdletBinding()]
 	Param (
 		[Parameter(ValueFromPipeline=$true,
 			ValueFromPipelineByPropertyName=$false)]
 		[String[]]$ComputerNames = $env:ComputerName,
 		[Parameter(ValueFromPipeline=$false,
 			ValueFromPipelineByPropertyName=$false)]
 		[String]$OutputPath
 	)
 	Begin {
 		Function check-ping {
 			Param (
 				$ComputerName
 			)
 			Process {
 				[regex]$regex = "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}"
 				if ($regex.Matches($ComputerName)){
 					$ipaddress = $ComputerName
 					$DNSinfo = $null
 				} else {
 					try {
 						$DNSinfo = [System.Net.Dns]::GetHostByName($ComputerName)
 					}
 					catch {
 						$message = "No DNS record found for {0}" -f $ComputerName
 						Write-Warning -Message $message
 						$DNSinfo = $null
 					}
 				}
 				if ($DNSinfo -ne $null){
 					$ping = New-Object System.Net.NetworkInformation.Ping
 					$pingOptions = New-Object System.Net.NetworkInformation.PingOptions
 					$pingOptions.DontFragment = $true
 					$buffer = [System.Text.Encoding]::ASCII.GetBytes("aaaaaaaaaaaaaaaaa")
 					$timeout = 120
 					$addresslist = $DNSinfo.AddressList
 					foreach ($ipaddress in $addresslist){
 						try {
 							$pingStatus = $ping.Send($ipaddress.IPAddressToString,$timeout,$buffer,$pingOptions)
 							if ($pingStatus.Status -eq "Success"){
 								Write-Output $true
 							} elseif ($pingStatus.Status -eq "Failure"){
 								Write-Output $false
 							} else {
 								Write-Output $false
 							}
 						}
 						Catch {
 							$message = "Failed to ping {0}" -f $ComputerName
 							Write-Warning -Message $message
 							Write-Output $false
 						}
 					}
 				} elseif ($ipaddress -ne $null){
 					$ping = New-Object System.Net.NetworkInformation.Ping
 					$pingOptions = New-Object System.Net.NetworkInformation.PingOptions
 					$pingOptions.DontFragment = $true
 					$buffer = [System.Text.Encoding]::ASCII.GetBytes("aaaaaaaaaaaaaaaaa")
 					$timeout = 120
 					try {
 							$pingStatus = $ping.Send($ipaddress,$timeout,$buffer,$pingOptions)
 							if ($pingStatus.Status -eq "Success"){
 								Write-Output $true
 							} elseif ($pingStatus.Status -eq "Failure"){
 								Write-Output $false
 							} else {
 								Write-Output $false
 							}
 						}
 						Catch {
 							$message = "Failed to ping {0}" -f $ComputerName
 							Write-Warning -Message $message
 							Write-Output $false
 						}
 				} else {
 					Write-Output $false
 				}
 			}
 		}
 		Function workgetcomputerinfo {
 			Param (
 				[String]$ComputerName,
 				[String]$wmiQuery,
 				[switch]$TestWMI
 			)
 			Process {
 				$NameSpace = "Root\CIMV2"
 				$wmi = [WMISearcher]""
 				Switch ($ComputerName){
 					$env:ComputerName	{$wmi.scope.path = "$NameSpace"}
 					Default				{$wmi.scope.path = "\\$ComputerName\$NameSpace"}
 				}
 				#$wmi.scope.path = "\\$ComputerName\$NameSpace"
 				$wmi.query = $wmiQuery
 				TRY {
 					$canconnectwmi=$True
 					$WMIResult = $wmi.Get()
 				}
 				Catch {
 					Write-Warning "WMI failed to connect to $($ComputerName.ToUpper())"
 					$canconnectwmi = $false
 				}
 				if ($TestWMI){
 					Write-Output $canconnectwmi
 				} else {
 					Write-Output $WMIResult
 				}
 			}
 		}
 		Function Get-LocalGroupMember {
 			Param(
 				[String]$ComputerName,
 				$Groups
 			)
 			Begin {
 				$NameSpace = "Root\CIMV2"
 				$wmi = [WMISearcher]""
 				$wmi.options.timeout = "0:0:30"
 			}
 			Process {
 				Switch ($ComputerName){
 					$env:ComputerName	{$wmi.scope.path = "$NameSpace"}
 					Default				{$wmi.scope.path = "\\$ComputerName\$NameSpace"}
 				}
 				Foreach ($G in $GroupData){
 					$Name = $G.Name
 					$GroupUserQuery = "Select * from Win32_GroupUser where GroupComponent=`'Win32_Group.Domain=`"$ComputerName`",Name=`"$Name`"`'"
 					$wmi.query = $GroupUserQuery
 					$WMIResult = $wmi.Get()
 					if ($WMIResult -ne $null){
 						$Members = Foreach ($u in $WMIResult){
 							$user = ($u.PartComponent).Split("""")
 							$member = $user[1] + "\" + $user[3]
 							Write-Output $member
 						}
 						$data = @{'GroupName'=$Name;'Members'=$Members}
 						$LocalGroupMembers = New-Object -TypeName PSObject -Property $data
 						Write-Output $LocalGroupMembers
 					}
 				}
 			}
 		}
 		Function Get-RemoteProgram {
 			[CmdletBinding()]
 			param(
 				[string]$ComputerName,
 				[string]$OSType
 			)
 			Switch ($ComputerName){
 				$env:ComputerName	{$RegBase = [Microsoft.Win32.Registry]::LocalMachine}
 				Default				{$RegBase = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey([Microsoft.Win32.RegistryHive]::LocalMachine,$ComputerName)}
 			}
 			If ($OSType -eq "x86-based PC"){
 				$keypaths = 'SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall'
 			} else {
 				$keypaths = 'SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall','SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall'
 			}
 			Foreach ($Path in $keypaths){
 				try {
 					$RegUninstall = $RegBase.OpenSubKey($Path)
 					$RegUninstall.GetSubKeyNames() | 
 					ForEach-Object {
 						$DisplayName = ($RegBase.OpenSubKey("$Path\$_")).GetValue('DisplayName')
 						if ($DisplayName) {
 							$DisplayVersion = ($RegBase.OpenSubKey("$Path\$_")).GetValue('DisplayVersion')
 							$InstallLocation = ($RegBase.OpenSubKey("$Path\$_")).GetValue('InstallLocation')
 							$Publisher = ($RegBase.OpenSubKey("$Path\$_")).GetValue('Publisher')
 							New-Object -TypeName PSCustomObject -Property @{
 								Name = $DisplayName;
 								Version = $DisplayVersion;
 								InstallLocation = $InstallLocation;
 								Publisher = $Publisher
 							}
 						}
 					}
 				}
 				catch {
 					Write-Output $null
 				}
 			}
 		}
 		Function Get-installedComponents {
 			[CmdletBinding()]
 			param(
 				[string]$ComputerName,
 				[string]$OSType
 			)
 			
 			if ((Get-WmiObject -ComputerName $ComputerName -Class Win32_OperatingSystem -Property Caption |Select-Object -ExpandProperty Caption) -like "*2003*"){
 				Switch ($ComputerName){
 					$env:ComputerName	{$RegBase = [Microsoft.Win32.Registry]::LocalMachine}
 					Default				{$RegBase = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey([Microsoft.Win32.RegistryHive]::LocalMachine,$ComputerName)}
 				}
 				If ($OSType -eq "x86-based PC"){
 					$keypaths = 'SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\Oc Manager\Subcomponents'
 				} else {
 					$keypaths = 'SOFTWARE\Microsoft\Windows\CurrentVersion\Setup\Oc Manager\Subcomponents','SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Setup\Oc Manager\Subcomponents'
 				}
 				Foreach ($Path in $keypaths){
 					try {
 						$Reg_Key = $RegBase.OpenSubKey($Path)
 						$names = $Reg_Key.GetValueNames() 
 						Foreach ($Name in $names){
 							$value = $Reg_Key.GetValue($Name)
 							if ($value -gt 0){
 								$components_installed +=@($Name)
 							}
 						}
 						$Reg_Key.Close()
 						$RegBase.Close()
 						
 					}
 					catch {
 						Write-Output $null
 					}
 				}
 			} else {
 				$features = Get-WmiObject -Class Win32_ServerFeature -ComputerName $ComputerName
 				foreach ($feature in $features){
 					$components_installed +=@($feature.name)
 				}
 			}
 			Write-Output ($components_installed |Sort-Object)
 		}
 		Function Get-SNMPSettings {
 			[CmdletBinding()]
 			param(
 				[string]$ComputerName
 			)
 			Switch ($ComputerName){
 				$env:ComputerName	{$RegBase = [Microsoft.Win32.Registry]::LocalMachine}
 				Default				{$RegBase = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey([Microsoft.Win32.RegistryHive]::LocalMachine,$ComputerName)}
 			}
 			$PermittedManagers = 'SYSTEM\CurrentControlSet\services\SNMP\Parameters\PermittedManagers'
 			$ValidCommunities = 'SYSTEM\CurrentControlSet\services\SNMP\Parameters\ValidCommunities'
 			try {
 				$Reg = $RegBase.OpenSubKey($PermittedManagers)
 				$pManager = $Reg.GetValueNames()|foreach ($_){$Reg.GetValue($_)}
 				if ($pManager.Count -gt 1){
 					$pManager = [System.String]::Join(", ",$pManager)
 				}
 				$Reg = $RegBase.OpenSubKey($ValidCommunities)
 				$Communities = $Reg.GetValueNames()
 				$vCommunities = foreach ($community in $Communities){
 					$permission = Switch ($Reg.GetValue($community)){
 						1 {"None"}
 						2 {"Notify"}
 						4 {"Read"}
 						8 {"Write"}
 						16 {"Create"}
 					}
 					$data = "$community - $permission"
 					Write-output $data
 				}
 				if ($vCommunities.Count -gt 1){
 					$vCommunities = [System.String]::Join("; ",$vCommunities)
 				}
 				$data = @{'CommunityPermission'=$vCommunities;
 					'PermittedManagers'=$pManager}
 				$SNMPSETTINGS = New-Object -TypeName PSObject -Property $data
 				Write-Output $SNMPSETTINGS
 			}
 			catch {
 				Write-Output $null
 			}
 		}
 		Function Start-extCMD {
 			Param (
 				$cmdexe,
 				$Arguments
 			)
 			Begin {}
 			Process {
 				$Startinfo = New-Object System.Diagnostics.ProcessStartinfo
 				$Startinfo.FileName = $cmdexe
 				$Startinfo.Arguments = $Arguments
 				$Startinfo.UseShellExecute = $false
 				$Startinfo.CreateNoWindow = $true
 				$Startinfo.RedirectStandardError = $true
 				$Startinfo.RedirectStandardOutput = $true
 				$process = New-Object System.Diagnostics.Process
 				$process.StartInfo = $Startinfo
 				$process.Start() | Out-Null
 				$stdout = $process.StandardOutput.ReadToEnd()
 				$stderr = $process.StandardError.ReadToEnd()
 				if ($stderr -eq ""){
 					Write-output $stdout
 				} else {
 					Write-output $stderror
 				}
 			}
 			End {}
 		}
 		Function Get-SchTasks {
 			Param (
 				$ComputerName
 			)
 			Begin {}
 			Process {
 				Switch ($ComputerName){
 					$env:ComputerName	{$arg = "/Query /V /FO LIST"}
 					Default				{$arg = "/Query /S {0} /V /FO LIST" -f $ComputerName}
 				}
 				$tasklist = Start-extCMD -cmdexe schtasks -Arguments $arg
 				$taskarray = $tasklist.Split("`n")
 				foreach ($line in $taskarray){
 					if ($line.StartsWith("HostName:")){
 						$data = @{}
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("TaskName:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$tname = $item[1].Split("\")
 						$value = $tname.Get($tname.Count -1)
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Next Run Time:")){
 						$item = $line.Split(":")
 						$title = "{0}" -f ($item[0].Trim()).Replace(" ","")
 						$value = "{0}" -f ($line.TrimStart("Next Run Time:")).Trim()
 					}elseif ($line.StartsWith("Status:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Logon Mode:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Last Run Time:")){
 						$item = $line.Split(":")
 						$title = "{0}" -f ($item[0].Trim()).Replace(" ","")
 						$value = "{0}" -f ($line.TrimStart("Last Run Time:")).Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Last Result:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Author:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Task To Run:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Start In:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Comment:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Scheduled Task State:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Idle Time:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Power Management:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Run As User:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Delete Task If Not Rescheduled:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Stop Task If Runs X Hours and X Mins:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Schedule:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Schedule Type:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Start Time:")){
 						$item = $line.Split(":")		
 						$title = "{0}" -f ($item[0].Trim()).Replace(" ","")
 						$value = "{0}" -f ($line.TrimStart("Start Time:")).Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Start Date:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("End Date:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Days:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Months:")){
 						$item = $line.Split(":")
 						$title = ($item[0].Trim()).Replace(" ","")
 						$value = $item[1].Trim()
 						$data.Add($title,$value)
 					}elseif($line.StartsWith("Repeat: Every:")){ 
 						$item = $line.Split(":")
 						$title = "{0}{1}" -f ($item[0].Trim()).Replace(" ",""),($item[1].Trim()).Replace(" ","")
 						$value = $item[2].Trim()
 					}elseif ($line.StartsWith("Repeat: Until: Time:")){
 						$item = $line.Split(":")
 						$title = "{0}{1}{2}" -f ($item[0].Trim()).Replace(" ",""),($item[1].Trim()).Replace(" ",""),($item[2].Trim()).Replace(" ","")
 						$value = $item[3].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Repeat: Until: Duration:")){
 						$item = $line.Split(":")
 						$title = "{0}{1}{2}" -f ($item[0].Trim()).Replace(" ",""),($item[1].Trim()).Replace(" ",""),($item[2].Trim()).Replace(" ","")
 						$value = $item[3].Trim()
 						$data.Add($title,$value)
 					}elseif ($line.StartsWith("Repeat: Stop If Still Running:")){
 						$item = $line.Split(":")
 						$title = "{0}{1}" -f ($item[0].Trim()).Replace(" ",""),($item[1].Trim()).Replace(" ","")
 						$value = $item[2].Trim()
 						$data.Add($title,$value)
 						$task = New-Object -TypeName PSObject -Property $data
 						Write-Output $task
 					}
 				}
 			}
 			End {}
 		}
 		function getip($ComputerName) {
 			if(!(Test-Connection -ComputerName $ComputerName -Count 2 -Quiet)){
 				$IPaddresses = [System.Net.Dns]::GetHostAddresses($ComputerName) | Select-Object -ExpandProperty IPAddressToString
 				Foreach ($IP in $IPaddresses){
 					if (Test-Connection -ComputerName $IP -Count 1 -Quiet){
 						Write-Output $IP
 					}
 				}
 			}
 		}
 	}
 	Process {
 		Foreach ($ComputerName in $ComputerNames){
 			if (($ComputerName -eq "") -or ($ComputerName -eq $null) -or ($ComputerName -eq $env:ComputerName) -or ($ComputerName -eq ".")){
 				$ComputerName = $env:ComputerName
 			}
 			$ReportTitle = $ComputerName
 			$OutputPath = $OutputPath.TrimEnd("\")
 			$outfile = $OutputPath + "\" + $ComputerName + ".html"
 			$BIOSQuery = 'Select BiosCharacteristics, SMBIOSBIOSVersion, SMBIOSMajorVersion, SMBIOSMinorVersion, Version, SerialNumber from Win32_BIOS'
 			$CDROMDriveQuery = 'Select Drive, Manufacturer, Name from Win32_CDROMDrive'
 			$ComputerSystemQuery = 'Select * from Win32_ComputerSystem'
 			#$ComputerSystemQuery = 'Select Domain, DomainRole, Name, TotalPhysicalMemory,SystemType, Manufacturer, Model, Status, NumberOfLogicalProcessors, NumberOfProcessors from Win32_ComputerSystem'
 			$ComputerSystemProductQuery = 'Select Vendor, Name, IdentifyingNumber from Win32_ComputerSystemProduct'
 			$DiskDriveQuery = 'Select * From Win32_Logicaldisk Where Drivetype=3'
 			$GroupQuery = "Select Name from Win32_Group Where Domain=`'$ComputerName`'"
 			$IP4RouteTableQuery = 'Select Destination, Mask, NextHop from Win32_IP4RouteTable'
 			$NetworkAdapterConfigurationQuery = 'SELECT * FROM Win32_NetworkAdapterConfiguration WHERE IPEnabled = True'
 			$NTEventLogFileQuery = 'Select LogFileName, MaxFileSize, Name, OverwritePolicy from Win32_NTEventLogFile'
 			$OperatingSystemsQuery = 'Select * from Win32_OperatingSystem'
 			#$OperatingSystemsQuery = 'Select CSName,Caption, CSDVersion, InstallDate, LastBootupTime, OSLanguage, Version, WindowsDirectory, OSArchitecture from Win32_OperatingSystem'
 			$PageFileQuery = 'Select Drive, InitialSize, MaximumSize from Win32_PageFile'
 			$PhysicalMemoryQuery = 'Select BankLabel, Capacity, FormFactor, MemoryType from Win32_PhysicalMemory'
 			$PrinterQuery = 'Select DriverName, Name, PortName from Win32_Printer Where ServerName = Null'
 			$ProcessQuery = 'Select Caption, ExecutablePath from Win32_Process'
 			$ProcessorQuery = 'Select Description, ExtClock, L2CacheSize, Name, MaxClockSpeed, SocketDesignation from Win32_Processor'
 			$ProductQuery = 'Select Name, Vendor, Version, InstallDate from Win32_Product WHERE Name <> Null'
 			$RegistryQuery = 'Select CurrentSize, MaximumSize from Win32_Registry'
 			$ServicesQuery = "Select Name, DisplayName, Started, StartMode, StartName, State, PathName from Win32_Service"
 			$ShareQuery = 'Select Name, Description, Path, Type from Win32_Share'
 			$SoundDeviceQuery = 'Select Name, Manufacturer from Win32_SoundDevice'
 			$StartupCommandQuery = 'Select Command, Name, User from Win32_StartupCommand'
 			$SystemEnclosureQuery = 'Select ChassisTypes from Win32_SystemEnclosure'
 			$TapeDriveQuery = 'Select Name, Description, Manufacturer from Win32_TapeDrive'
 			$TimeZoneQuery = 'Select Description from Win32_TimeZone'
 			$QuickFixEngineeringQuery = "Select Description, HotFixID, InstalledOn from Win32_QuickFixEngineering Where HotfixID <> 'File 1' And HotfixID <> 'Q147222'"
 			$UserAccountQuery = "Select Description, Name from Win32_UserAccount Where Domain=`'$ComputerName`'"
 			$VideoControllerQuery = 'Select AdapterCompatibility, AdapterRAM, Name from Win32_VideoController'
 			Try {
 				$test = $true
 				if (($ComputerName -ne "") -or ($ComputerName -ne $null) -or ($ComputerName -ne $env:ComputerName) -or ($ComputerName -ne ".")){
 					$pingResult = check-ping -ComputerName $ComputerName
 				} else {
 					$pingResult = $true
 				}
 				if ($pingResult){
 					$test = workgetcomputerinfo -computername $ComputerName -wmiQuery $OperatingSystemsQuery -TestWMI
 				} else {
 					$test = $false
 				}
 			}
 			Catch {
 				Write-Warning "WMI failed to connect to $($computername.ToUpper())"
 				$test = $false
 			}
 			if ($test){
 				$Process = workgetcomputerinfo -computername $ComputerName -wmiQuery $ProcessQuery
 				#$Product = workgetcomputerinfo -computername $ComputerName -wmiQuery $ProductQuery
 				$Registry = workgetcomputerinfo -computername $ComputerName -wmiQuery $RegistryQuery
 				$SystemEnclosure = workgetcomputerinfo -computername $ComputerName -wmiQuery $SystemEnclosureQuery
 				$TapeDrive = workgetcomputerinfo -computername $ComputerName -wmiQuery $TapeDriveQuery
 				$snmp = Get-SNMPSettings -ComputerName $ComputerName
 				$schtsks = Get-SchTasks -ComputerName $ComputerName
 				$installedComponents = Get-installedComponents -ComputerName $ComputerName -OSType $OSType 
 				$OperatingSystemsData = $OperatingSystems | Select @{Name="Name";Expression={$_.CSName}},
 						@{Name="OS";Expression={$_.Caption}},
 						@{Name="ServicePack";Expression={$_.CSDVersion}},
 						@{Name="SystemLevel";Expression={$_.Version}},
 						@{Name="InstallDate";Expression={$_.InstallDate}},
 						@{Name="WindowsDirectory";Expression={$_.WindowsDirectory}},
 						@{Name="LastBoot";Expression={$_.ConvertToDateTime($_.LastBootupTime)}},
 						@{Name="Uptime";Expression={(Get-Date) - ($_.ConvertToDateTime($_.LastBootupTime))}}
 				$BIOSData = $BIOS | Select-Object @{Name="SerialNumber";Expression={$_.SerialNumber}},@{Name="SMBIOSBIOSVersion";Expression={$_.SMBIOSBIOSVersion}},
 						@{Name="SMBIOSMajorVersion";Expression={$_.SMBIOSMajorVersion}},
 						@{Name="SMBIOSMinorVersion";Expression={$_.SMBIOSMinorVersion}},
 						@{Name="Version";Expression={$_.Version}}
 				$ProcessorData = $Processor | Select-Object @{Name="Name";Expression={$_.Name}},
 						@{Name="Description";Expression={$_.Description}},
 						@{Name="L2CacheSize";Expression={$_.L2CacheSize}},
 						@{Name="MaxClockSpeed";Expression={$_.MaxClockSpeed}},
 						@{Name="SocketDesignation";Expression={$_.SocketDesignation}}
 				$ShareData = $Share | Select-Object @{Name="Name";Expression={$_.Name}},
 						@{Name="Description";Expression={$_.Description}},
 						@{Name="Path";Expression={$_.Path}}
 				$ComputerSystemData= $ComputerSystem | Select-Object @{Name="Domain";Expression={$_.Domain}},
 						@{Name="DomainRole";Expression={$_.DomainRole}},
 						@{Name="Manufacturer";Expression={$_.Manufacturer}},
 						@{Name="Model";Expression={$_.Model}},
 						@{Name="SystemType";Expression={$_.SystemType}},
 						@{Name="Name";Expression={$_.Name}},
 						@{Name="Status";Expression={$_.Status}},
 						@{Name="TotalPhysicalMemory";Expression={$_.TotalPhysicalMemory}},
 						@{Name="TotalPhysicalMemoryGB";Expression={"{0:N0}" -f ($_.TotalPhysicalMemory /1GB)}},
 						@{Name="NumberofProcessors";Expression={$_.NumberofProcessors}},
 						@{Name="NumberofLogicalProcessors";Expression={$_.NumberofLogicalProcessors}}
 				$CDROMDriveData = $CDROMDrive | Select-Object @{Name="Drive";Expression={$_.Drive}},
 						@{Name="Manufacturer";Expression={$_.Manufacturer}},
 						@{Name="Name";Expression={$_.Name}}
 				$ComputerSystemProductData = $ComputerSystemProduct | Select-Object @{Name="Manufacturer";Expression={$_.Vendor}},
 						@{Name="IdentifyingNumber";Expression={$_.IdentifyingNumber}},
 						@{Name="Name";Expression={$_.Name}}
 				$IP4RouteTable = $IP4RouteTable | Select-Object @{Name="Destination";Expression={$_.Destination}},
 						@{Name="Mask";Expression={$_.Mask}},
 						@{Name="NextHop";Expression={$_.NextHop}}
 				$NetworkAdapterConfigurationData = $NetworkAdapterConfiguration | Select-Object @{Name="Description";Expression={$_.Description}},
 						@{Name="IPAddress";Expression={$_.IPAddress}},
 						@{Name="MACAddress";Expression={$_.MACAddress}},
 						@{Name="DNSHostName";Expression={$_.DNSHostName}},
 						@{Name="DNSDomain";Expression={$_.DNSDomain}},
 						@{Name="DNSDomainSuffixSearchOrder";Expression={$_.DNSDomainSuffixSearchOrder}},
 						@{Name="WINSPrimaryServer";Expression={$_.WINSPrimaryServer}},
 						@{Name="WINSSecondaryServer";Expression={$_.WINSSecondaryServer}}
 				$NTEventLogFileData = $NTEventLogFile | Select-Object @{Name="LogFileName";Expression={$_.LogFileName}},
 						@{Name="MaxFileSize";Expression={$_.MaxFileSize}},
 						@{Name="Name";Expression={$_.Name}},
 						@{Name="OverwritePolicy";Expression={$_.OverwritePolicy}}
 				$PhysicalMemory = $PhysicalMemoryinfo | Select-Object @{Name="BankLabel";Expression={$_.BankLabel}},
 						@{Name="CapacityGB";Expression={$_.Capacity / 1GB}},
 						@{Name="FormFactor";Expression={$_.FormFactor}},
 						@{Name="MemoryType";Expression={$_.MemoryType}}
 				$Printer = $Printer	| Select-Object @{Name="DriverName";Expression={$_.DriverName}},
 						@{Name="Name";Expression={$_.Name}},
 						@{Name="PortName";Expression={$_.PortName}}
 				$servicedata = $Services | Select-Object @{Name="ServiceName";Expression={$_.Name}},
 						@{Name="ServiceDisplayName";Expression={$_.DisplayName}},
 						@{Name="StartMode";Expression={$_.StartMode}},
 						@{Name="Started";Expression={$_.Started}},
 						@{Name="StartName";Expression={$_.StartName}},
 						@{Name="State";Expression={$_.State}},
 						@{Name="PathName";Expression={$_.PathName}}
 				$Processdata = $Process | Select-Object @{Name="ProcessName";Expression={$_.Caption}},
 						@{Name="ExecutablePath";Expression={$_.ExecutablePath}}
 				$schtaskdata = $schtsks |Select-Object @{Name="TaskName";Expression={$_.TaskName}},
 						@{Name="Command";Expression={$_."TaskToRun"}},
 						@{Name="RunAs";Expression={$_."RunAsUser"}},
 						@{Name="ScheduleStart";Expression={(("{0} {1} {2} {3}" -f $_.StartTime,$_.StartDate,$_.Days,$_.Months).Replace("N/A"," "))}},
 						@{Name="Repeat";Expression={(("{0} {1} {2} {3}" -f $_.RepeatEvery,$_.RepeatUntilTime,$_.RepeatUntilDuration,$_.RepeatStopIfStillRunning).Replace("N/A"," "))}},
 						@{Name="State";Expression={$_."ScheduledTaskState"}}
 				$diskData = $DiskDrive | Select DeviceID,@{Name="SizeGB";Expression={$_.size/1GB -as [int]}},
 						@{Name="FreeGB";Expression={"{0:N2}" -f ($_.Freespace/1GB)}},
 						@{Name="PercentFree";Expression={"{0:P2}"  -f ($_.Freespace/$_.Size)}}
 				$startupData = $StartupCommand | Select-Object @{Name="Name";Expression={$_.Name}},
 						@{Name="Command";Expression={$_.Command}},
 						@{Name="User";Expression={$_.User}}
 				$TimeData = $TimeZone | Select-Object @{Name="Name";Expression={$_.Description}}
 				$network = $NetworkAdapterConfiguration | Select-Object @{Name="IPAddress";Expression={$_.IPAddress}}
 				if ($network.Count -gt 1){
 					$IP = getip($ComputerName)
 					Foreach ($n in $Network){
 						if ($n.IPAddress -eq $IP){$network = $IP}
 					}
 				}
 				$fragments=@()
 				$ReportTitle = $ComputerName
 	$head = @"
 	<meta name="viewport" content="width=device-width, initial-scale=1.0">
 	<style>
 	body {
 		font-family:Arial;
 		font-size:12px;
 	}
 	header {
 		width:98%;
 		color:white;
 		margin-right:auto;
 		margin-left:auto;
 		padding:2px;
 		overflow:hidden;
 	}
 	h1 {
 		font-size:2em;
 		margin:0;
 		padding:0;
 		padding-top:5px;
 	}
 	h2 {
 		font-size:1.6;
 	}
 	h3 {
 		font-size:1.4;
 	}
 	h4 {
 		font-size:1.1;
 	}
 	hgroup {
 		width:90%;
 		float:left;
 	}
 	hgroup h1 {
 		Padding-top:0;
 		line-height:1em;
 	}
 	hgroup h2 {
 		text-transform:uppercase;
 		font-size:1.2em;
 		line-height:1em;
 	}
 	nav li {
 		margin-right:2px;
 		float: center;
 		display: inline;
 	}
 	nav ul {
 		list-style-type:none;
 		overflow: hidden;
 	}
 	section {
 		width:98%;
 		background-color:White;
 		margin-right:auto;
 		margin-left:auto;
 		padding:5px;
 	}
 	p {
 		margin:6px;
 	}
 	.processordata , .physicalMem , .storage, .cdrom, .ip4routing, .ntevent, .printer, .product, .service,.schtask, .process, .quickfix {
 		margin:0px;padding:0px;
 		width:100%;
 		-moz-border-radius-bottomleft:5px;
 		-webkit-border-bottom-left-radius:5px;
 		border-bottom-left-radius:5px;
 		-moz-border-radius-bottomright:5px;
 		-webkit-border-bottom-right-radius:5px;
 		border-bottom-right-radius:5px;
 		-moz-border-radius-topright:5px;
 		-webkit-border-top-right-radius:5px;
 		border-top-right-radius:5px;
 		-moz-border-radius-topleft:5px;
 		-webkit-border-top-left-radius:5px;
 		border-top-left-radius:5px;
 	}
 	table {
 		border-collapse: collapse;
 		border-spacing: 0;
 		width:100%;
 		height:100%;
 		margin:0px;padding:0px;
 	}
 	th{
 		text-align:center;
 		border-width:0px 0px 1px 1px;
 		font-size:14px;
 		font-family:Arial;
 		font-weight:bold;
 	}
 	th:first-child {
 		-moz-border-radius-topleft:5px;
 		-webkit-border-top-left-radius:5px;
 		border-top-left-radius:5px;
 		border-width:0px 0px 1px 0px;
 	}
 	th:last-child {
 		-moz-border-radius-topright:5px;
 		-webkit-border-top-right-radius:5px;
 		border-top-right-radius:5px;
 		border-width:0px 0px 1px 1px;
 	}
 	tr:last-child td:first-child{
 		-moz-border-radius-bottomleft:5px;
 		-webkit-border-bottom-left-radius:5px;
 		border-bottom-left-radius:5px;
 	}
 	tr:last-child td:last-child {
 		-moz-border-radius-bottomright:5px;
 		-webkit-border-bottom-right-radius:5px;
 		border-bottom-right-radius:5px;
 		border-width:0px 0px 0px 0px;
 	}
 	tr:nth-child(odd){
 	}
 	tr:nth-child(even){
 	}
 	td{
 		vertical-align:middle;
 		border-width:0px 1px 1px 0px;
 		text-align:left;
 		padding:2px;
 		font-size:11px;
 		font-family:Arial;
 		font-weight:normal;
 	}
 	tr:last-child td{
 		border-width:0px 1px 0px 0px;
 	}
 	tr td:last-child{
 		border-width:0px 0px 1px 0px;
 	}
 	footer {
 		width:80%;
 		background-color:black;
 		color:white;
 		margin-right:auto;
 		margin-left:auto;
 		height:50px;
 		padding:10px;
 	}
 
 
 	/* 
 	Max width before this PARTICULAR table gets nasty
 	This query will take effect for any screen smaller than 760px
 	and also iPads specifically.
 	*/
 	@media all and (min-width: 760px){
 	.nontable {
 		-webkit-column-count: 2;
 		-moz-column-count: 2;
 		column-count: 2;
 		-webkit-column-gap: 15px;
 		-moz-column-gap: 15px;
 		column-gap: 40px;
 	}
 	}
 	@media only screen and (max-width: 760px),
 	(min-device-width: 768px) and (max-device-width: 1024px) {
 		/* Force table to not be like tables anymore */
 		.processordata , .physicalMem , .storage, .cdrom, .ip4routing, .ntevent, .printer, .product, .service, .process, .quickfix {
 		}
 		table, thead, tbody, th, td, tr { 
 			display: block; 
 		}
 		/* Hide table headers (but not display: none;, for accessibility) */
 		thead tr, th { 
 			position: absolute;
 			top: -9999px;
 			left: -9999px;
 		}
 		tr {
 		}
 		td { 
 			/* Behave  like a "row" */
 			border: none;
 			position: relative;
 			padding-left: 50%; 
 		}
 		td:before { 
 			/* Now like a table header */
 			position: absolute;
 			/* Top/left values mimic padding */
 			top: 6px;
 			left: 6px;
 			width: 45%; 
 			padding-right: 10px; 
 			white-space: wrap;
 		}
 		/* Label the data */
 		.processordata td:nth-of-type(1):before { content: "Name"; }
 		.processordata td:nth-of-type(2):before { content: "Description"; }
 		.processordata td:nth-of-type(3):before { content: "L2 Cache Size"; }
 		.processordata td:nth-of-type(4):before { content: "Max Clock Speed"; }
 		.processordata td:nth-of-type(5):before { content: "Socket Designation"; }
 		.physicalMem td:nth-of-type(1):before { content: "Bank Lable"; }
 		.physicalMem td:nth-of-type(2):before { content: "Capacity GB"; }
 		.physicalMem td:nth-of-type(3):before { content: "FormFactor"; }
 		.physicalMem td:nth-of-type(4):before { content: "MemoryType"; }
 		.storage td:nth-of-type(1):before { content: "DeviceID"; }
 		.storage td:nth-of-type(2):before { content: "SizeGB"; }
 		.storage td:nth-of-type(3):before { content: "FreeGB"; }
 		.storage td:nth-of-type(4):before { content: "PercentFree"; }
 		.cdrom td:nth-of-type(1):before { content: "Drive"; }
 		.cdrom td:nth-of-type(2):before { content: "Manufacturer"; }
 		.cdrom td:nth-of-type(3):before { content: "Name"; } 
 		.ip4routing td:nth-of-type(1):before { content: "Destination"; }
 		.ip4routing td:nth-of-type(2):before { content: "Mask"; }
 		.ip4routing td:nth-of-type(3):before { content: "NextHop"; }
 		.ntevent td:nth-of-type(1):before { content: "LogFileName"; }
 		.ntevent td:nth-of-type(2):before { content: "MaxFileSize"; }
 		.ntevent td:nth-of-type(3):before { content: "Name"; }
 		.ntevent td:nth-of-type(4):before { content: "Overwrite Policy"; }
 		.printer td:nth-of-type(1):before { content: "Driver Name"; }
 		.printer td:nth-of-type(2):before { content: "Name"; }
 		.printer td:nth-of-type(3):before { content: "Port Name"; }
 		.product td:nth-of-type(1):before { content: "Name"; }
 		.product td:nth-of-type(2):before { content: "Version"; }
 		.product td:nth-of-type(3):before { content: "Install Location"; }
 		.product td:nth-of-type(4):before { content: "Publisher"; }
 		.service td:nth-of-type(1):before { content: "Service Name"; }
 		.service td:nth-of-type(2):before { content: "Start Mode"; }
 		.service td:nth-of-type(3):before { content: "State"; }
 		.service td:nth-of-type(4):before { content: "Start Name"; }
 		.service td:nth-of-type(5):before { content: "Path Name"; }
 		.schtask td:nth-of-type(1):before { content: "Task Name"; }
 		.schtask td:nth-of-type(2):before { content: "Command"; }
 		.schtask td:nth-of-type(3):before { content: "Run As"; }
 		.schtask td:nth-of-type(4):before { content: "Schedule Start"; }
 		.schtask td:nth-of-type(5):before { content: "Repeat"; }
 		.schtask td:nth-of-type(6):before { content: "State"; }
 		.process td:nth-of-type(1):before { content: "Process Name"; }
 		.process td:nth-of-type(2):before { content: "Executable Path"; }
 		.quickfix td:nth-of-type(1):before { content: "Hotfix ID"; }
 		.quickfix td:nth-of-type(2):before { content: "Description"; }
 		.quickfix td:nth-of-type(3):before { content: "Installed On"; }
 	}
 
 	/* Smartphones (portrait and landscape) ----------- */
 	@media only screen
 	and (min-device-width : 320px)
 	and (max-device-width : 480px) {
 		body { 
 			padding: 0; 
 			margin: 0; 
 			width: 320px;
 		}
 	}
 
 	/* iPads (portrait and landscape) ----------- */
 	@media only screen and (min-device-width: 768px) and (max-device-width: 1024px) {
 		body { 
 			width: 495px; 
 		}
 	}
 	</style>
 	<Title>$ReportTitle</Title>
 	"@
 	$header = @"
 	<header>
 	  <hgroup>
 		<H1>System Report</H1>
 		<H4>$(Get-Date -DisplayHint Date | out-string)</H4>
 		<nav><ul>
 			<li><a href=#{0}_Summary>System Summary</a></li>
 			<li><a href=#{0}_Storage>Storage</a></li>
 			<li><a href=#{0}_Shares>Shares</a></li>
 			<li><a href=#{0}_ComputerSystemProductData>Computer System</a></li>
 			<li><a href=#{0}_InstalledComponents>Installed Components</a></li>
 			<li><a href=#{0}_StartupData>Startup</a></li>
 			<li><a href=#{0}_GroupUserData>Local Groups</a></li>
 			<li><a href=#{0}_localUserData>Local Users</a></li>
 			<li><a href=#{0}_IP4RouteTable>IP4 Route Table</a></li>
 			<li><a href=#{0}_NetworkAdapterConfigurationData>Network Adapter</a></li>
 			<li><a href=#{0}_NTEventLogFileData>NT Event Log</a></li>
 			<li><a href=#{0}_PhysicalMemory>Physical Memory</a></li>
 			<li><a href=#{0}_Processor>Processor</a></li>
 			<li><a href=#{0}_Printer>Printers</a></li>
 			<li><a href=#{0}_Product>Products</a></li>
 			<li><a href=#{0}_services>Services</a></li>
 			<li><a href=#{0}_schtasks>Scheduled Tasks</a></li>
 			<li><a href=#{0}_Processes>Processes</a></li>
 			<li><a href=#{0}_QuickFix>QuickFix</a></li>
 		</ul></nav>
 		</hgroup>
 	</header>
 	"@ -f $Computername.ToUpper()
 				$nav=""
 				$nav+="<nav><ul>"
 				$nav+=("<li><a href=#{0}_Summary>System Summary</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_Storage>Storage</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_Shares>Shares</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_ComputerSystemProductData>Computer System</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_InstalledComponents>Installed Components</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_StartupData>Startup</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_GroupUserData>Local Groups</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_localUserData>Local Users</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_IP4RouteTable>IP4 Route Table</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_NetworkAdapterConfigurationData>Network Adapter</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_NTEventLogFileData>NT Event Log</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_PhysicalMemory>Physical Memory</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_Processor>Processor</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_Printer>Printers</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_Product>Products</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_services>Services</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_schtasks>Scheduled Tasks</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_Processes>Processes</a></li>" -f $computername.ToUpper())
 				$nav+=("<li><a href=#{0}_QuickFix>QuickFix</a></li>" -f $computername.ToUpper())
 				$nav+="</ul></nav>"
 				$footer="Report v{3} run {0} by {1}\{2}" -f (Get-Date),$env:USERDOMAIN,$env:USERNAME,$reportVersion
 				$head+=("<a name='{0}_Summary'>{0}</a> " -f $computername.ToUpper())
 				$head+="<br><hr>"
 				$fragments+=$header
 				#$fragments+=$nav
 				#$fragments+=("<H2><a name='{0}_Summary'>System Summary</a></H2>" -f $computername.ToUpper())
 				$fragments+="<section>"
 				$fragments+=("<H2>System Summary</H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"nontable`">"
 				$fragments+="<p><strong>Name:</strong> {0}    </p><p><strong>IP Address:</strong> {1}</p>" -f $OperatingSystemsData.Name,$network.IPAddress
 				$fragments+="<p><strong>OS:</strong> {0} {1}</p>" -f $OperatingSystemsData.OS,$OperatingSystemsData.ServicePack
 				$fragments+="<p><strong>SystemLevel:</strong> {0}</p>" -f $OperatingSystemsData.SystemLevel
 				$fragments+="<p><strong>WindowsDirectory:</strong> {0}</p>" -f $OperatingSystemsData.WindowsDirectory
 				$fragments+="<p><strong>LastBoot:</strong> {0} </p><p><strong>Uptime:</strong> {1}</p>" -f $OperatingSystemsData.LastBoot,$OperatingSystemsData.Uptime
 				
 				$fragments+="<p><strong>Domain:</strong> {0}  </p><p><strong>DomainRole:</strong> {1}</p>" -f $ComputerSystemData.Domain,$ComputerSystemData.DomainRole
 				$fragments+="<p><strong>Manufacturer:</strong> {0} </p><p><strong> Model:</strong> {1}</p>" -f $ComputerSystemData.Manufacturer,$ComputerSystemData.Model
 				$fragments+="<p><strong>SystemType:</strong> {0}  </p><p><strong>Status:</strong> {1}</p>" -f $ComputerSystemData.SystemType,$ComputerSystemData.Status
 				$fragments+="<p><strong>TimeZone:</strong> {0}</p>" -f $TimeData.Name
 				$fragments+="<p><strong>TotalPhysicalMemory:</strong> {0}  </p><p><strong>TotalPhysicalMemoryGB:</strong> {1}</p>" -f $ComputerSystemData.TotalPhysicalMemory,$ComputerSystemData.TotalPhysicalMemoryGB
 				$fragments+="<p><strong>Number of Physical Processors:</strong> {0}  </p><p><strong>Number of Logical Processors:</strong> {1}</p>" -f $ComputerSystemData.NumberOfProcessors,$ComputerSystemData.NumberOfLogicalProcessors
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_ComputerSystemProductData'>Computer System</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"nontable`">"
 				$fragments+="<p><strong>Name:</strong> {0}  </p><p><strong>Manufacturer:</strong> {1}</p>" -f $ComputerSystemProductData.Name,$ComputerSystemProductData.Manufacturer
 				$fragments+="<p><strong>IdentifyingNumber:</strong> {0} </p><p>" -f $ComputerSystemProductData.IdentifyingNumber
 				$fragments+="<strong>SerialNumber:</strong> {0}  </p><p><strong>Version:</strong> {1}</p>" -f $BIOSData.SerialNumber,$BIOSData.Version
 				$fragments+="<p><strong>SMBIOSBIOSVersion:</strong> {0}</p><p><strong>SMBIOSMajorVersion:</strong> {1}  </p><p><strong>SMBIOSMinorVersion:</strong> {2}</p>" -f $BIOSData.SMBIOSBIOSVersion,$BIOSData.SMBIOSMajorVersion,$BIOSData.SMBIOSMinorVersion
 				$fragments+="<p><strong>SoundCard Name:</strong> {0}  </p><p><strong>SoundCard Manufacturer:</strong> {1}</p>" -f $SoundDevice.Name ,$SoundDevice.Manufacturer 
 				Foreach ($VidAdapter in $VideoController){
 				$fragments+="<p><strong>Video Controller Name:</strong> {0}  </p><p><strong>Video Adapter Compatibility:</strong> {1}  </p><p><strong>Video Adapter RAM:</strong> {2}</p>" -f $VidAdapter.Name ,$VidAdapter.AdapterCompatibility ,$VidAdapter.RAM 
 				}
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_Processor'>Processor</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"processordata`">"
 				$fragments+=$ProcessorData | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_PhysicalMemory'>Physical Memory</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"physicalMem`">"
 				$fragments+=$PhysicalMemory | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_Storage'>Storage</a></H2>" -f $computername.ToUpper())
 				$fragments+="<H4>Local Storage</H4>"
 				$fragments+="<div class=`"storage`">"
 				$fragments+=$diskData | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+="<H4>CDROM Drive</a></H4>"
 				$fragments+="<div class=`"cdrom`">"
 				$fragments+=$CDROMDriveData | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_NetworkAdapterConfigurationData'>Network Adapters</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"nontable`">"
 				Foreach ($nic in $NetworkAdapterConfigurationData){
 					$fragments+="<p><strong>Description: {0}</strong></p>" -f $nic.Description
 					$fragments+="<p><strong>IPAddress:</strong> {0}  </p><p><strong>MACAddress:</strong> {1}</p>" -f $nic.IPAddress,$nic.MACAddress
 					$fragments+="<p><strong>DNSHostName:</strong> {0}  </p><p><strong>DNSDomain:</strong> {1}</p>" -f $nic.DNSHostName,$nic.DNSDomain
 					$fragments+="<p><strong>DNSDomainSuffixSearchOrder:</strong> {0}</p>" -f $nic.DNSDomainSuffixSearchOrder
 					$fragments+="<p><strong>WINSServers:</strong> {0}, {1}</p>" -f $nic.WINSPrimaryServer,$nic.WINSSecondaryServer
 				}
 				if ($snmp -ne $null){
 					$fragments+="<p><strong>SNMP Communities with permission:</strong> {0}  </p><p><strong>SNMP Managers:</strong> {1}</p>" -f $snmp.CommunityPermission,$snmp.PermittedManagers
 				}
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_IP4RouteTable'>IP4 Route Table</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"ip4routing`">"
 				$fragments+=$IP4RouteTable | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_InstalledComponents'>Installed Components</a></H2>" -f $computername.ToUpper())
 				$fragments+="<p><ul>"
 				Foreach ($item in $installedComponents){
 					$fragments+= "<li>{0}</li>" -f $item
 				}
 				$fragments+="</ul></p>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_GroupUserData'>Local Groups</a></H2>" -f $computername.ToUpper())
 				Foreach ($Gp in $GroupUser){
 					$fragments+="<p><strong>{0}</strong><br>" -f $Gp.GroupName
 					$fragments+="<ul>"
 					$fragments+= Foreach ($gmem in $Gp.Members){"<li>{0}</li>" -f $gmem}
 					$fragments+="</ul></p>"
 				}
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_localUserData'>Local Users</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"nontable`">"
 				Foreach ($localUser in $UserAccount){
 					$fragments+="<p><strong>Name:</strong> {0}     <strong>Description:</strong> {1} </p>" -f $localUser.Name,$localUser.Description
 				}
 				$fragments+="<br>"
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_NTEventLogFileData'>NT Event Log</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"ntevent`">"
 				$fragments+=$NTEventLogFileData | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_Shares'>Shares</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"nontable`">"
 				Foreach($share in $ShareData){
 					$fragments+="<p><strong>Name:</strong> {0}  <br><strong>Description:</strong> {1}  <br><strong>Path:</strong> {2}</p>" -f $share.Name,$share.Description,$share.Path
 				}
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_Printer'>Printers</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"printer`">"
 				$fragments+=$Printer | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_Product'>Products</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"product`">"
 				$fragments+=$Product |Select-Object Name,Version,InstallLocation,Publisher | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_StartupData'>Startup</a></H2>" -f $computername.ToUpper())
 				Foreach ($Startu in $startupData){
 					$fragments+="<p><strong>{0}</strong><br>" -f $Startu.Name
 					$fragments+="<ul>"
 					$fragments+= "<li>{0}</li>" -f $Startu.User
 					$fragments+= "<li>{0}</li>" -f $Startu.Command
 					$fragments+="</ul></p>"
 				}
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_services'>Services</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"service`">"
 				$fragments+=$servicedata | Select-Object ServiceName,StartMode,State,StartName,PathName | Sort-Object State,StartMode,ServiceName | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_schtasks'>Scheduled Tasks</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"schtask`">"
 				$fragments+=$schtaskdata | Select-Object TaskName,Command,RunAs,ScheduleStart,Repeat,State | Sort-Object State,TaskName | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_Processes'>Processes</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"process`">"
 				$fragments+=$Processdata | Select-Object ProcessName,ExecutablePath | Sort-Object ExecutablePath | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$fragments+="<section>"
 				$fragments+=("<H2><a name='{0}_QuickFix'>QuickFix</a></H2>" -f $computername.ToUpper())
 				$fragments+="<div class=`"quickfix`">"
 				$fragments+=$QuickFixEngineering | Select-Object HotfixID,Description,InstalledOn | Sort-Object InstalledOn | ConvertTo-Html -Fragment
 				$fragments+="</div>"
 				$fragments+=("<p><a href=#{0}_Summary>Back to System Summary</a></p>" -f $computername.ToUpper())
 				$fragments+="</section>"
 				$compinfo = ConvertTo-Html -Head $head -Title $ReportTitle -PreContent ($fragments | out-String) -PostContent "<br><I>$footer</I>" 
 				$compinfo | Out-File $outfile
 			}
 		}
 	}
 }
`

