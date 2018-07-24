---
Author: alexander petrovskiy                                                                              #
Publisher: alexander petrovskiy, softwaretestingusingpowershell.wordpress.com                                #
Copyright: © 2011 alexander petrovskiy, softwaretestingusingpowershell.wordpress.com. all rights reserved.   #
Email: 
Version: 4.1
Encoding: utf-8
License: cc0
PoshCode ID: 2907
Published Date: 2011-08-10t03
Archived Date: 2011-09-15t09
---

# esximgmt module - 

## Description

how can you automate your esxi tasks having only bare esxi software? may cmdlets in such case don’t work or work with serious limitations. to fulfill though partially the lack of ‘bare esxi’ management tools, the esximgmt module has been written and tested.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 #######################################################################################################################
 #######################################################################################################################
 Set-StrictMode -Version Latest
 
 if ((Get-PSSnapin VMware.VimAutomation.Core) -ne $null -and `
 	(Get-PSSnapin VMware.VimAutomation.Core).Name.Length -gt 0)
 {
 	Remove-PSSnapin VMware.VimAutomation.Core;
 }
 Add-PSSnapin VMware.VimAutomation.Core;
 if ($Env:PROCESSOR_ARCHITECTURE -eq 'x86'){
 	. "$env:ProgramFiles\VMware\Infrastructure\vSphere PowerCLI\Scripts\Initialize-PowerCLIEnvironment.ps1"
 }
 if ($Env:PROCESSOR_ARCHITECTURE -eq 'AMD64'){
 	. "${env:ProgramFiles(x86)}\VMware\Infrastructure\vSphere PowerCLI\Scripts\Initialize-PowerCLIEnvironment.ps1"
 }
 
 if (Test-Path -Path ($PSScriptRoot + '\plink.exe')){
 	[string]$script:plinkPath = $PSScriptRoot + '\plink.exe';
 }
 
 function Get-CurrentTime
 	<#
 		.SYNOPSIS
 			The Get-CurrentTime function is used to write in the timestamp in the log file.
 
 		.DESCRIPTION
 			The Get-CurrentTime functions is used for getting the current time of operation. 
 			As s time source used [System.DateTime]::Now.TimeOfDay property.
 
 		.EXAMPLE
 			PS C:\> Get-CurrentTime
 
 		.OUTPUTS
 			System.String
 	#>
 {	$timeOfDay = [System.DateTime]::Now.TimeOfDay;
 	$time = "$($timeOfDay.Hours):$($timeOfDay.Minutes):$($timeOfDay.Seconds)`t";
 	return $time;}
 function Connect-ESXi
 	<#
 		.SYNOPSIS
 			Connects to a single ESXi you are planning to work with.
 
 		.DESCRIPTION
 			The Connect-ESXi function is intended to be the first the user runs 
 			while working with this module.
 
 		.PARAMETER  Server
 			The name of IP address of the server you work with. 
 			It's also used further as an user@server combination.
 
 		.PARAMETER  Port
 			Default value is 443. See the Connect-VIServer description.
 
 		.PARAMETER  Protocol
 			Default value is HTTPS. See the Connect-VIServer description.
 			
 		.PARAMETER  User
 			Username that is often root. It's also used further as an user@server combination.
 			
 		.PARAMETER  Password
 			Password to connect to a ESXi. It's also used as a parameter for plink.
 			
 		.PARAMETER  DatastoreName
 			The name of the datastore you work with.
 			
 		.PARAMETER  Drive
 			The name that will represents the content of the datastore.
 			
 		.EXAMPLE
 			PS C:\> Connect-ESXi -Server 192.168.1.1 -Port 443
 					-Protocol HTTPS -User root -Password 123
 					-DatastoreName datastore1 
 					-Drive server1
 
 		.INPUTS
 			System.String, System.Int32
 
 		.OUTPUTS
 			None
 
 	#>
 {
 	[CmdletBinding()]
 	param(
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Server,
 		  [Parameter(Mandatory=$false)]
 		  [int]$Port = 443,
 		  [Parameter(Mandatory=$false)]
 		  [string]$Protocol = 'HTTPS',
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$User,
 		  [AllowEmptyString()]
 		  [string]$Password = '',
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$DatastoreName,
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Drive
 		  )
 	[string]$script:pwd = $Password;
 		  
 	[VMware.VimAutomation.ViCore.Impl.V1.VIServerImpl]$script:esxiserver = `
 		Connect-VIServer -Server $Server `
 			-User $User -Password $Password `
 			-Protocol $Protocol -Port $Port;
 	[VMware.VimAutomation.ViCore.Impl.V1.Inventory.VMHostImpl]$script:vmhost = `
 		Get-VMHost -Server $script:esxiserver;
 
 	[VMware.VimAutomation.ViCore.Impl.V1.DatastoreManagement.DatastoreImpl]$script:datastore = `
 		Get-Datastore -Server $script:esxiserver -Name $DatastoreName;
 	try{New-DatastoreDrive -Datastore $script:datastore -Name $Drive;
 	[string]$script:dsdrive = $Drive;}catch{}
 }
 function Disconnect-ESXi
 	<#
 		.SYNOPSIS
 			Disconnects the latest connected ESXi server.
 
 		.DESCRIPTION
 			The function cleans up the variable that stores VIServer object of the server you connected.
 
 		.EXAMPLE
 			PS C:\> Disconnect-ESXi
 
 		.INPUTS
 			None
 
 		.OUTPUTS
 			None
 
 	#>
 {
 	Disconnect-VIServer -Server $script:esxiserver -Force:$true;
 }
 function Invoke-ESXiCommand
 	<#
 		.SYNOPSIS
 			Runs a command or a semicolon-separated sequence of commands on an ESXi server.
 
 		.DESCRIPTION
 			This function runs plink.exe with the command supplied and optionally 
 			suppressses the console window.
 			
 		.PARAMETER  Server
 			The name of IP address of the server you work with. 
 			It's also used further as an user@server combination.
 			
 		.PARAMETER  User
 			Username that is often root. It's also used further as an user@server combination.
 			
 		.PARAMETER  Password
 			Password to connect to a ESXi. It's also used as a parameter for plink.
 
 		.PARAMETER  Command
 			The string that plink.exe runs on a server.
 
 		.PARAMETER  PathToPlink
 			Used if for some reason you don't want to put plink.exe in the module folder.
 
 		.PARAMETER  ShowWindow
 			Enables or disables appearance of the plink.exe console window.
 			
 		.PARAMETER  OperationTimeout
 			The time that the module sleeps before start next operation.
 
 		.EXAMPLE
 			PS C:\> Invoke-ESXiCommand -Server 192.168.1.1 `
 					 -User root -Password 123 `
 					 -Command 'ls ~; sleep 10s; exit;' `
 					 -PathToPlink 'C:\plink.exe' `
 					 -ShowWindow $true -OperationTimeout 10;
 
 		.INPUTS
 			System.String, System.Int32, System.Boolean
 
 		.OUTPUTS
 			None
 
 	#>
 {
 	[CmdletBinding()]
 	param(
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Server,
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$User,
 		  [AllowEmptyString()]
 		  [string]$Password = '',
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Command,
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$PathToPlink,
 		  [Parameter(Mandatory=$false)]
 		  [bool]$ShowWindow = $true,
 		  [Parameter(Mandatory=$false)]
 		  [int]$OperationTimeout = 2
 		 )
 	try{
 		[string[]]$private:connectArgs = `
 			@(
 			  "-ssh $($User)@$($Server) -pw $($Password) $($Command) "
 			  );
 		if ($ShowWindow){
 			Start-Process -FilePath $PathToPlink `
 				-ArgumentList $private:connectArgs;
 		}
 		else{
 			Start-Process -FilePath $PathToPlink `
 				-ArgumentList $private:connectArgs -NoNewWindow;
 		}
 		sleep -Seconds $OperationTimeout;
 	}catch{};
 }
 function New-ESXiFSDirectory
 	<#
 		.SYNOPSIS
 			Creates a directory on the datastore file system.
 
 		.DESCRIPTION
 			This function creates a directory and is a cane.
 
 		.PARAMETER  Server
 			The name of IP address of the server you work with. 
 			It's also used further as an user@server combination.
 			
 		.PARAMETER  User
 			Username that is often root. It's also used further as an user@server combination.
 			
 		.PARAMETER  Password
 			Password to connect to a ESXi. It's also used as a parameter for plink.
 			
 		.PARAMETER  Path
 			The absolute path to the folder which a new folder will be created within.
 
 		.PARAMETER  Name
 			The name for a new folder.
 
 		.PARAMETER  PathToPlink
 			Used if for some reason you don't want to put plink.exe in the module folder.
 
 		.PARAMETER  OperationTimeout
 			The time that the module sleeps before start next operation.
 
 		.EXAMPLE
 			PS C:\> New-ESXiFSDirectory -Server 192.168.1.1 `
 					-User root -Password 123 `
 					-Path "/vmfs/volumes/datastore3" -Name foldername `
 					-PathToPlink C:\plink.exe -OperationTimeout 20;
 
 		.INPUTS
 			System.String, System.Int32
 
 		.OUTPUTS
 			None
 
 	#>
 {
 	[CmdletBinding()]
 	param(
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Server,
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$User,
 		  [AllowEmptyString()]
 		  [string]$Password = '',
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Path,
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Name,
 		  [Parameter(Mandatory=$false)]
 		  [AllowEmptyString()]
 		  [string]$PathToPlink,
 		  [int]$OperationTimeout = 2
 		 )
 	if ($PathToPlink.Length -lt 1){$PathToPlink = $script:plinkPath;}
 	[string]$private:commandToRun = "`"cd $($Path);mkdir $($Name);exit;`"";
 	Invoke-ESXiCommand -Server $Server -User $User -Password $Password `
 		-Command $private:commandToRun -PathToPlink $PathToPlink `
 		-ShowWindow $false -OperationTimeout $OperationTimeout;
 }
 function Register-ESXiVM
 	<#
 		.SYNOPSIS
 			Register an VMX file.
 
 		.DESCRIPTION
 			this function register a new virtual machine specified as a full path to its .vmx file.
 
 		.PARAMETER  Server
 			The name of IP address of the server you work with. 
 			It's also used further as an user@server combination.
 			
 		.PARAMETER  User
 			Username that is often root. It's also used further as an user@server combination.
 			
 		.PARAMETER  Password
 			Password to connect to a ESXi. It's also used as a parameter for plink.
 
 		.PARAMETER  Path
 			The full path to a VMX file.
 
 		.PARAMETER  PathToPlink
 			Used if for some reason you don't want to put plink.exe in the module folder.
 			
 		.PARAMETER  OperationTimeout
 			The time that the module sleeps before start next operation.
 
 		.EXAMPLE
 			PS C:\> Register-ESXiVM -Server 192.168.1.1 `
 						-User root -Password 123 `
 						-Path "/vmfs/volumes/datastore3/vmname/vmname.vmx" `
 						-PathToPlink C:\plink.exe;
 
 		.INPUTS
 			System.String, System.Int32
 
 		.OUTPUTS
 			System.String
 
 	#>
 {
 	[CmdletBinding()]
 	param(
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Server,
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$User,
 		  [AllowEmptyString()]
 		  [string]$Password = '',
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Path,
 		  [Parameter(Mandatory=$false)]
 		  [AllowEmptyString()]
 		  [string]$PathToPlink,
 		  [int]$OperationTimeout = 20
 		 )
 	if ($PathToPlink.Length -lt 1){$PathToPlink = $script:plinkPath;}
 	[string]$private:commandToRun = "`"/bin/vim-cmd solo/registervm $($Path);exit;`"";
 	Invoke-ESXiCommand -Server $Server -User $User -Password $Password `
 		-Command $private:commandToRun -PathToPlink $PathToPlink `
 		-ShowWindow $false -OperationTimeout $OperationTimeout;
 }
 function Start-ESXiVM
 	<#
 		.SYNOPSIS
 			Starts a powered off orsuspended virtual machine.
 
 		.DESCRIPTION
 			This function is intended to start a new virtual machine as well as an existing one.
 			In case the machine is generated from an image, it also answer the question whether
 			the machine was copied or moved.
 
 		.PARAMETER  Server
 			The name of IP address of the server you work with. 
 			It's also used further as an user@server combination.
 			
 		.PARAMETER  User
 			Username that is often root. It's also used further as an user@server combination.
 			
 		.PARAMETER  Password
 			Password to connect to a ESXi. It's also used as a parameter for plink.
 
 		.PARAMETER  Id
 			The Id of a virtual machine that is generated by the server that hosts it.
 
 		.PARAMETER  PathToPlink
 			Used if for some reason you don't want to put plink.exe in the module folder.
 			
 		.PARAMETER  OperationTimeout
 			The time that the module sleeps before start next operation.
 
 		.EXAMPLE
 			PS C:\> Start-ESXiVM -Server 192.168.1.1 `
 						-User root -Password 123 `
 						-Id 504 `
 						-PathToPlink C:\plink.exe;
 
 		.INPUTS
 			System.String, System.Int32
 
 		.OUTPUTS
 			None
 
 	#>
 {
 	[CmdletBinding()]
 	param(
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Server,
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$User,
 		  [AllowEmptyString()]
 		  [string]$Password = '',
 		  [Parameter(Mandatory=$true)]
 		  [int]$Id,
 		  [Parameter(Mandatory=$false)]
 		  [AllowEmptyString()]
 		  [string]$PathToPlink,
 		  [int]$OperationTimeout = 10
 		 )
 	if ($PathToPlink.Length -lt 1){$PathToPlink = $script:plinkPath;}
 	[string]$private:commandToRun = "`"/bin/vim-cmd vmsvc/power.on $($Id.ToString());exit;`"";
 	Invoke-ESXiCommand -Server $Server -User $User -Password $Password `
 		-Command $private:commandToRun -PathToPlink $PathToPlink `
 		-ShowWindow $true -OperationTimeout $OperationTimeout;
 	try{Get-VM -Id $Id;
 		if ((Get-VM -Id $Id).PowerState -eq `
 		[VMware.VimAutomation.ViCore.Types.V1.Inventory.PowerState]::PoweredOff){
 		[string]$private:commandToRun = "`"/bin/vim-cmd vmsvc/message $($Id.ToString()) 0 2;exit;`"";
 		Invoke-ESXiCommand -Server $Server -User $User -Password $Password `
 			-Command $private:commandToRun -PathToPlink $PathToPlink `
 			-ShowWindow $true -OperationTimeout 2;}
 	}catch{
 		[string]$private:commandToRun = "`"/bin/vim-cmd vmsvc/message $($Id.ToString()) 0 2;exit;`"";
 		Invoke-ESXiCommand -Server $Server -User $User -Password $Password `
 			-Command $private:commandToRun -PathToPlink $PathToPlink `
 			-ShowWindow $true -OperationTimeout 2;
 	}
 }
 function Stop-ESXiVM
 	<#
 		.SYNOPSIS
 			Powers off (or what is set for this option on your server) a virtual machine.
 
 		.DESCRIPTION
 			This funciton simply 'presses' the red button.
 
 		.PARAMETER  Server
 			The name of IP address of the server you work with. 
 			It's also used further as an user@server combination.
 			
 		.PARAMETER  User
 			Username that is often root. It's also used further as an user@server combination.
 			
 		.PARAMETER  Password
 			Password to connect to a ESXi. It's also used as a parameter for plink.
 
 		.PARAMETER  Id
 			The Id of a virtual machine that is generated by the server that hosts it.
 
 		.PARAMETER  PathToPlink
 			Used if for some reason you don't want to put plink.exe in the module folder.
 			
 		.PARAMETER  OperationTimeout
 			The time that the module sleeps before start next operation.
 
 		.EXAMPLE
 			PS C:\> Stop-ESXiVM -Server 192.168.1.1 `
 						-User root -Password 123 `
 						-Id (Get-ESXiVMId $vm);
 
 		.INPUTS
 			System.String, System.Int32
 
 		.OUTPUTS
 			None
 
 	#>
 {
 	[CmdletBinding()]
 	param(
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Server,
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$User,
 		  [AllowEmptyString()]
 		  [string]$Password = '',
 		  [Parameter(Mandatory=$true)]
 		  [int]$Id,
 		  [Parameter(Mandatory=$false)]
 		  [AllowEmptyString()]
 		  [string]$PathToPlink,
 		  [int]$OperationTimeout = 60
 		 )
 	if ($PathToPlink.Length -lt 1){$PathToPlink = $script:plinkPath;}
 	[string]$private:commandToRun = "`"/bin/vim-cmd vmsvc/power.off $($Id.ToString());exit;`"";
 	Invoke-ESXiCommand -Server $Server -User $User -Password $Password `
 		-Command $private:commandToRun -PathToPlink $PathToPlink `
 		-ShowWindow $true -OperationTimeout $OperationTimeout;
 }
 function Suspend-ESXiVM
 	<#
 		.SYNOPSIS
 			Puts a virtual machine into a suspended state.
 
 		.DESCRIPTION
 			This function 'presses' the yellow button.
 
 		.PARAMETER  Server
 			The name of IP address of the server you work with. 
 			It's also used further as an user@server combination.
 			
 		.PARAMETER  User
 			Username that is often root. It's also used further as an user@server combination.
 			
 		.PARAMETER  Password
 			Password to connect to a ESXi. It's also used as a parameter for plink.
 
 		.PARAMETER  Id
 			The Id of a virtual machine that is generated by the server that hosts it.
 
 		.PARAMETER  PathToPlink
 			Used if for some reason you don't want to put plink.exe in the module folder.
 			
 		.PARAMETER  OperationTimeout
 			The time that the module sleeps before start next operation.
 
 		.EXAMPLE
 			PS C:\> Suspend-ESXiVM -Server 192.168.1.1 `
 						-User root -Password 123 `
 						-Id $private:vmId `
 						-PathToPlink C:\plink.exe;
 
 		.INPUTS
 			System.String, System.Int32
 
 		.OUTPUTS
 			None
 
 	#>
 {
 	[CmdletBinding()]
 	param(
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$Server,
 		  [Parameter(Mandatory=$true)]
 		  [ValidateNotNullOrEmpty()]
 		  [string]$User,
 		  [AllowEmptyString()]
 		  [string]$Password = '',
 		  [Parameter(Mandatory=$true)]
 		  [int]$Id,
 		  [Parameter(Mandatory=$false)]
 		  [AllowEmptyString()]
 		  [string]$PathToPlink,
 		  [int]$OperationTimeout = 60
 		 )
 	if ($PathToPlink.Length -lt 1){$PathToPlink = $script:plinkPath;}
 	[string]$private:commandToRun = "`"/bin/vim-cmd vmsvc/power.suspend $($Id.ToString());exit;`"";
 	Invoke-ESXiCommand -Server $Server -User $User -Password $Password `
 		-Command $private:commandToRun -PathToPlink $PathToPlink `
 		-ShowWindow $true -OperationTimeout $OperationTimeout;
 }
 function Get-ESXiVMId
 	<#
 		.SYNOPSIS
 			Returns the Id of a virtual machine.
 
 		.DESCRIPTION
 			This function receives a virtual machine object and extract the Id property.
 
 		.PARAMETER  VM
 			The VMware.VimAutomation.ViCore.Impl.V1.Inventory.VirtualMachineImpl object.
 
 		.EXAMPLE
 			PS C:\> [int]$private:vmId = `
 						Get-ESXiVMId -VM (Get-VM -Name $VMName);
 
 		.INPUTS
 			VMware.VimAutomation.ViCore.Impl.V1.Inventory.VirtualMachineImpl
 
 		.OUTPUTS
 			System.Int32
 
 	#>
 (
 	[CmdletBinding()]
 	[OutputType([System.Int32])]
 	[Parameter(Mandatory=$true)]
 	[ValidateNotNullOrEmpty()]
 	[VMware.VimAutomation.ViCore.Impl.V1.Inventory.VirtualMachineImpl]$VM
 )
 {
 	#[int]$private:vmId = [System.Text.RegularExpressions.Regex]::Match( `
 	[int]$private:vmId = [System.Text.RegularExpressions.Regex]::Match( `
 			$VM.Id.ToString(), "(?<=[-]).*").Value;
 	return $private:vmId;
 }
 function Get-ESXiVMName
 	<#
 		.SYNOPSIS
 			Returns the name of a virtual machine that contains a guest with name given.
 
 		.DESCRIPTION
 			This function enumerates all the virtual machines hosted on a server 
 			in order to get the name that has the corresponding virtual machine.
 
 		.PARAMETER  VMHostname
 			Ths name(s) of the guest host(s).
 
 		.EXAMPLE
 			PS C:\> $vmname = Get-ESXiVMName -VMHostname 'B45E19A64B5E418'
 
 		.INPUTS
 			System.String[]
 
 		.OUTPUTS
 			System.String
 
 	#>
 (
 	[CmdletBinding()]
 	[OutputType([System.String])]
 	[Parameter(Mandatory=$true)]
 	[ValidateNotNullOrEmpty()]
 	[string[]]$VMHostname
 )
 {
 	[System.Collections.ArrayList]$vmnames = `
 		New-Object System.Collections.ArrayList;
 	
 	[VMware.VimAutomation.ViCore.Impl.V1.Inventory.VirtualMachineImpl[]]$vmss = `
 		Get-VM *;
 
 	for ([int]$private:i = 0; $private:i -lt $VMHostname.Length; $private:i++)
 	{
 		[bool]$addEmptyName = $true;
 		foreach($vm in $vmss)
 		{
 			if ($vm.Name -ne $null -and `
 				$vm.Guest -ne $null -and `
 				$vm.Guest.HostName -ne $null -and `
 				$vm.Guest.HostName.Length -gt 0){
 				if ($vm.Guest.HostName.toUpper() -eq `
 					$VMHostname[$private:i].toUpper() -or `
 					$vm.Guest.HostName.toUpper().Contains($VMHostname[$private:i].toUpper()))
 				{
 					$null = $vmnames.Add($vm.Name);
 					$addEmptyName = $false;
 					break;
 				}
 			}
 		}
 		if ($addEmptyName){$vmnames.Add("");}
 	}
 	return $vmnames;
 }
 function New-ESXiVMs
 	<#
 		.SYNOPSIS
 			Generates a bunch of virtual machines.
 
 		.DESCRIPTION
 			This function generates new virtual machine by copying the existing virtual machine files,
 			replacing the name of the original machine with the name that's provided and
 			copying the result to the server the module connected..
 			
 		.PARAMETER  TemplateVMName
 			The name of the virtual machine used as a template.
 			
 		.PARAMETER  Count
 			The number of virtual machines you need.
 			
 		.PARAMETER  Logname
 			The full path to the log file.
 			
 		.PARAMETER  NewVMName
 			The base name of new virtual machines. Virtual machines will be named in the follwing order:
 			newvm_1
 			newvm_2
 			...
 			newvm_20.
 
 		.PARAMETER  BasePath
 			The path in the local file system where the template resides. the hardcoded values are
 			basepath\hdd - the folder where the virtual disk(s) 
 			<templatevm>-flat.vmdk,
 			<templatevm>-<number>delta.vmdk, 
 			<templatevm>-Snapshot<number>.vmsn (snapshot of a machine that was turned on)
 			are stored. In short, the files of megabyte of gigabyte size.
 			basepath\template - the folder where the rest of files 
 			(namely, 
 			<templatevm>.vmx, 
 			<templatevm>.vmxf, 
 			<templatevm>.vmsd, 
 			<templatevm>.vmdk (the header),
 			<templatevm>-<number>.vmdk,
 			<templatevm>-Snapshot<number>.vmsn (snapshot of a machine that was turned off)
 			) reside, in other words, the files the size of which
 			are measured in kilobytes
 
 		.PARAMETER  PathToPlink
 			Used if for some reason you don't want to put plink.exe in the module folder.
 			
 		.PARAMETER  OperationTimeout
 			The time that the module sleeps before start next operation.
 			
 		.EXAMPLE
 			PS C:\> New-ESXiVMs -TemplateVMName 'template XP SP2 Sv' -Count 20 `
 						-Logname "C:\VMTests\20VMs.txt" -NewVMName 'XPSP2a' `
 						-BasePath 'C:\VMTests\xpsp2';
 
 		.INPUTS
 			System.String, System.Int32
 
 		.OUTPUTS
 			None
 
 	#>
 (
 	[CmdletBinding()]
 	[Parameter(Mandatory=$true)]
 	[ValidateNotNullOrEmpty()]
 	[string]$TemplateVMName,
 	[Parameter(Mandatory=$true)]
 	[int]$Count = 1,
 	[Parameter(Mandatory=$true)]
 	[ValidateNotNullOrEmpty()]
 	[string]$Logname,
 	[Parameter(Mandatory=$true)]
 	[string]$NewVMName = 'newVM_',
 	[Parameter(Mandatory=$true)]
 	[ValidateNotNullOrEmpty()]
 	[string]$BasePath,
 	[Parameter(Mandatory=$false)]
 	[AllowEmptyString()]
 	[string]$PathToPlink,
 	[int]$OperationTimeout = 600
 )
 {
 	if ($PathToPlink.Length -lt 1){$PathToPlink = $script:plinkPath;}
 	#[VMware.VimAutomation.ViCore.Impl.V1.Inventory.VirtualMachineImpl]$script:templateVM = `
 	#{
 	#}
 
 	for ($private:i = 0; $private:i -lt $Count; $private:i++)
 	{
 		[string]$private:currentVMName = $NewVMName + ($private:i + 1).ToString();
 		"$(Get-CurrentTime)Start creating the virtual machine '$($private:currentVMName)'" >> $Logname; 
 		New-Item -Path "$($BasePath)\$($private:currentVMName)" -type directory -Force;
 		[string]$templateStorage = $BasePath + "\template";
 		[string]$hddStorage = $BasePath + "\hdd";
 		Get-ChildItem -LiteralPath $templateStorage | `
 		%{[string]$currentFile = $_.FullName; 
 			[string]$newFile = `
 				$currentFile.ToLower().Replace("$($templateStorage)\$($TemplateVMname)".ToLower(), `
 				"$($BasePath)\$($private:currentVMName)\$($private:currentVMName)");
 		"Changing $($currentFile) to $($newFile)" >> $Logname;
 		Copy-Item -Path $currentFile -Destination $newFile;
 			(Get-Content $newFile) | %{ $_ -replace $TemplateVMname, $private:currentVMName; } | `
 				Set-Content -Path $newFile;}
 		New-ESXiFSDirectory -Server $script:esxiserver.Name -User $script:esxiserver.User -Password $script:pwd `
 			-Path "/vmfs/volumes/$($script:datastore.Name)" -Name $private:currentVMName `
 			-PathToPlink $PathToPlink -OperationTimeout 20;
 		"$(Get-CurrentTime)Copying the config files '$($private:currentVMName)'" >> $Logname;
 		Copy-DatastoreItem -Item "$($BasePath)\$($private:currentVMName)\$($private:currentVMName)*.*" `
 			-Destination "$($script:dsdrive):\$($private:currentVMName)\";
 		"$(Get-CurrentTime)Copying the virtual drive image(s) '$($private:currentVMName)'" >> $Logname;
 		Get-ChildItem -Path "$($hddStorage)" | 
 			%{[string]$private:newFileName = $_.Name.Replace($TemplateVMname, $private:currentVMName);
 			Copy-DatastoreItem -Item $_.FullName `
 				-Destination "$($script:dsdrive):\$($private:currentVMName)\$($private:newFileName)";}
 		
 		"$(Get-CurrentTime)Registering the VM '$($private:currentVMName)'" >> $Logname;
 		Register-ESXiVM -Server $script:esxiserver.Name -User $script:esxiserver.User -Password $script:pwd `
 			-Path "/vmfs/volumes/$($script:datastore.Name)/$($private:currentVMName)/$($private:currentVMName).vmx" `
 			-PathToPlink $PathToPlink;
 
 		[int]$private:vmId = Get-ESXiVMId -VM (Get-VM -Name $private:currentVMName);
 		
 		
 		"$(Get-CurrentTime)Starting the VM '$($private:currentVMName)'" >> $Logname;
 		Start-ESXiVM -Server $script:esxiserver.Name -User $script:esxiserver.User -Password $script:pwd `
 			-Id $private:vmId `
 			-PathToPlink $PathToPlink;
 		
 		sleep -Seconds $OperationTimeout;
 		Suspend-ESXiVM -Server $script:esxiserver.Name -User $script:esxiserver.User -Password $script:pwd `
 			-Id $private:vmId `
 			-PathToPlink $PathToPlink;
 	}
 }
 
 Export-ModuleMember -Function Connect-ESXi, Invoke-ESXiCommand, New-ESXiFSDirectory; 
 Export-ModuleMember -Function Register-ESXiVM, Start-ESXiVM, Suspend-ESXiVM, New-ESXiVMs;
 Export-ModuleMember -Function Get-ESXiVMId, Get-ESXiVMName, Disconnect-ESXi, Stop-ESXiVM;
`

