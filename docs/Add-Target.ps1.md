---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 6.1
Encoding: utf-8
License: mitl
PoshCode ID: 3066
Published Date: 
Archived Date: 2012-02-05t01
---

#  - 

## Description

a module that wraps iscsicli.exe to provide basic iscsi management capabilities

## Comments



## Usage



## TODO



## module

`add-target`

## Code

`#
 #
 #
 #
 #
 #
 #
 #
 #
 #
 #
 
 
 ###########################################################
 
 #
 $payloadIPV4	 	= 1;
 $payloadFQDN 		= 2;
 $payloadIPV6		= 5;
 
 #
 $securityTunnelMode			= 0x40;
 $securityTransportMode		= 0x20;
 $securityPFSEnable			= 0x10;
 $securityAggressiveMode		= 0x08;
 $securityMainMode			= 0x04;
 $securityIPSECIKEEnabled	= 0x02;
 $securityValidFlags			= 0x01;
 
 #
 #
 $loginRequireIPSEC			= 0x01;
 $loginMultipathEnabled		= 0x02;
 
 #
 #
 #
 $authTypeNone		= 0;
 $authTypeChap		= 1;
 $authTypeMutualChap	= 2;
 
 #
 #
 $targetHideStaticTarget		= 0x02;
 $targetMergeTargetInfo		= 0x04;
 
 ###########################################################
 
 #
 #
 function Add-Target
 {
 	[CmdletBinding(SupportsShouldProcess=$true)]
 	param(
 		[Parameter(Mandatory=$true)]
 		[Alias("Name")]
 		[string]		
 		$targetName,
 
 		[Parameter(Mandatory=$true)]
 		[Alias("Address")]
 		[string]
 		$targetPortalAddress
 	)
 	process
 	{
 		& iscsicli qaddtarget $targetName, $targetPortalAddress;
 	}
 }
 
 #
 #
 function Remove-Target
 {
 <#
 	This command will remove a target from the list of persisted targets.
 
 Boot Configuration Known Issues (Windows Server 2003 Boot Initiator)
 The Microsoft iSCSI Software Initiator boot version GUI does not allow you to view which adapter is set to boot. In order 
 
 to determine which adapter the system is set to boot with, you can use the following command:
  From a command prompt  type �iscsibcg /showibf� to find the MAC address of the boot adapter 
  Then run the command �ipconfig /all�  
  Compare the MAC address of the adapter to those listed with ipconfig /all 
 
 MPIO Failover in an iSCSI boot configuration using the Microsoft iSCSI Software Initiator
 
 In Fail Over Only, no load balancing is performed. The primary path functions as the active path and all other paths are 
 
 standby paths. The active path is used for sending all I/O. If the active path fails,  one of the standby paths  becomes 
 
 the active path.   When the  formerly active path is reconnected, it becomes a standby path and  a "failback" does not 
 
 occur.   This behavior is due to Media Sensing is disabled by default in the boot version of the Microsoft iSCSI Software 
 
 Initiator and is by design.  However, the registry key can be changed to enable fail back.  For more information, please 
 
 see
      
 For more information: 
 239924  How to disable the Media Sensing feature for TCP/IP in Windows
 http://support.microsoft.com/default.aspx?scid=kb;EN-US;239924   
 #>
 	[CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact='high')]
 	param(
 		[Parameter(Mandatory=$true,ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
 		[Alias("Name")]
 		[string]
 		$targetName,
 		
 		[Parameter(ValueFromPipelineByPropertyName=$true)]
 		[Alias("Address")]
 		[string]
 		$targetPortalAddress,
 		
 		[Parameter(ValueFromPipelineByPropertyName=$true)]
 		[Alias('Port')]
 		[int]
 
 defined for use by iSCSI.
 		$TargetPortalSocket = 3260,
 		
 		[Parameter(ValueFromPipelineByPropertyName=$true)]
 		[Alias( "InitiatorInstanceName" )]
 		[string]
 
 initiator used is selected by the iSCSI initiator service.
 		$InitiatorName, 
 
 		[Parameter(ValueFromPipelineByPropertyName=$true)]
 		[string]
 
 specified then the kernel mode initiator driver chooses the initiator port used.
 		$InitiatorPort = '*',		
 		
 		[Parameter()]
 		[switch]
 		$persist,
 		
 		[Parameter()]
 		[switch]
 		$force		
 	)
 	
 	process
 	{
 		Write-Verbose "remove-target ...";
 		Write-Verbose "  TargetName: $targetName";
 		Write-Verbose "  TargetPortalAddress: $targetPortalAddress";
 		Write-Verbose "  TargetPortalSocket: $targetPortalSocket";
 		Write-Verbose "  InitiatorInstanceName: $InitiatorName";
 		Write-Verbose "  InitiatorPort: $initiatorPort";
 		Write-Verbose "  Persist: $persist";
 		Write-Verbose "  Force: $force";
 
 		if( -not ( $force -or $pscmdlet.ShouldProcess( $targetName, 'Remove iSCSI target' ) ) )
 		{
 			return;
 		}
 		if( $persist -and $InitiatorName )
 		{
 			$iscsi = "iscsicli removepersistenttarget $InitiatorName $targetName $InitiatorPort 
 
 $targetPortalAddress $TargetPortalSocket"
 			Write-Verbose $iscsi;
 			invoke-expression $iscsi
 		}
 		else
 		{
 			$iscsi = "iscsicli removetarget $targetName";
 			Write-Verbose $iscsi;
 			invoke-expression $iscsi
 		}
 		
 	}
 }
 
 #
 #
 function Add-TargetPortal
 {
 <#
 This command will add a target portal to the list of persisted target portals. The iSCSI initiator service will perform a 
 
 SendTargets operation to each target portal in the list whenever the service starts and whenever a full refresh of the 
 
 target list is requested. 
 
 #>
 	#[CmdletBinding(SupportsShouldProcess=$true)]
 	param(
 		[Parameter(Mandatory=$true)]
 		[Alias("Address")]
 		[string]
 		$targetPortalAddress,
 		
 		[Parameter()]
 		[string]
 
 specifying * for this parameter, the iSCSI initiator service will use the initiator node name as the CHAP username.
 		$username,
 		
 		[Parameter()]
 		[string]
 
 initiator will use this secret to compute a hash value based on the challenge sent by the target. 
 		$password
 	)
 	process
 	{
 		if( $username )
 		{
 			& iscsicli qaddtargetportal $targetPortalAddress $username $password;
 		}
 		else
 		{
 			& iscsicli qaddtargetportal $targetPortalAddress;
 		}
 	}
 	
 }
 
 #
 function Remove-TargetPortal
 {
 <#
 This command will remove a target portal from the list of persisted target  portals. The iSCSI initiator service will 
 
 perform a SendTargets operation to each target portal in the list whenever the service starts and whenever a full refresh 
 
 of the target list is requested. Note that the command does not purge the targets discovered via this target portal from 
 
 the list of targets maintained by the service.
 #>
 	[CmdletBinding(SupportsShouldProcess=$true)]
 	param(
 		[Parameter(Mandatory=$true,ValueFromPipeline=$true)]
 		[Alias('Name')]
 		[string]
 		$targetPortalAddress,
 		
 		[Parameter()]
 		[Alias('Port')]
 		[int]
 
 defined for use by iSCSI.
 		$TargetPortalSocket = 3260,
 
 		[Parameter()]
 		[string]
 
 initiator used is selected by the iSCSI initiator service.
 		$InitiatorName = '', 
 
 		[Parameter()]
 		[string]
 
 specified then the kernel mode initiator driver chooses the initiator port used.
 		$InitiatorPort = ''
 	)
 	
 	process
 	{
 		& iscsicli removetargetportal $targetPortalAddress $TargetPortalSocket $InitiatorName $InitiatorPort 
 	}
 }
 
 #
 function Update-TargetPortal
 {
 <#
 This command will perform a SendTargets operation to the target portal and include the discovered targets into the list of 
 
 targets maintained by the service. It does not add the target portal to the persistent list.
 #>
 
 	[CmdletBinding(SupportsShouldProcess=$true)]
 	param(
 		[Parameter(Mandatory=$true)]
 		[Alias("Address")]
 		[string]
 		$targetPortalAddress,
 
 		[Parameter()]
 		[Alias("Port")]
 		[int]
 
 defined for use by iSCSI.
 		$TargetPortalSocket = 3260,
 
 		[Parameter()]
 		[string]
 
 initiator used is selected by the iSCSI initiator service.
 		$InitiatorName, 
 
 		[Parameter()]
 		[int]
 
 specified then the kernel mode initiator driver chooses the initiator port used.
 		$InitiatorPort		
 	)
 	
 	process
 	{
 		& iscsicli refreshtargetportal $targetPortalAddress $TargetPortalSocket $InitiatorName $InitiatorPort
 	}
 }
 
 #
 #
 function Get-Target
 {
 <#
 	This command will display the list of persistent targets configured for all initiators.
 #>
 	[CmdletBinding(DefaultParameterSetName='Local')]
 	param(
 		[Parameter( ParameterSetName='Persistent' )]
 		[switch]
 		$persistent, 
 		[Parameter( ParameterSetName='Local' )]
 		[switch]
 		$force
 	)
 	
 	process
 	{
 		if( $persistent )
 		{
 			$data = & iscsicli ListPersistentTargets | Out-String;
 			
 			$data | Write-Verbose;
 			$data -replace "[`r`n]+","=" -split "==" | where { $_ -match ':\s+' } | foreach {			
 				$_ | convertFrom-iSCSIOutput
 			}
 		}
 		else
 		{
 			if( $force )
 			{
 				$data = & iscsicli ListTargets T
 			}
 			else
 			{
 				$data = & iscsicli ListTargets
 			}
 			
 			$data | Select-String '^\s+\S+:\S+$' | foreach{ $_ -replace '^\s+','' -replace '\s+$','' };		
 		}		
 	}
 
 }
 
 #
 function Get-TargetPortal
 {
 	[CmdletBinding()]
 	param()
 	process
 	{
 		$data = & iscsicli ListTargetPortals | Out-String;
 		$data | Write-Verbose;
 		$data -replace "[`r`n]+","=" -split "==" | where { $_ -match ':\s+' } | foreach {			
 			$_ | convertFrom-iSCSIOutput
 		}
 	}	
 }
 
 #
 function Get-TargetInfo
 {
 <#
 This command will return information about the target specified by TargetName. The iSCSI initiator service maintains a 
 
 separate set of information about every target organized by each mechanism by which it was discovered. This means that 
 
 each instance of a target can have different information such as target portal groups. Discovery Mechanism is an optional 
 
 parameter and if not specified then only the list of discovery mechanisms for the target are displayed. If Discovery 
 
 Mechanism is specified then information about the target instance discovered by that mechanism is displayed.
 #>
 	[CmdletBinding()]
 	param(
 		[Parameter(Mandatory=$true,ValueFromPipeline=$true)]
 		[string]
 		$targetName, 
 		[Parameter()]
 		[string]
 		$discoveryMechanism 
 	)
 	process
 	{
 	}
 }
 
 #
 #
 #
 function Connect-Target
 {
 <#
 This command will login to a target 
 #>
 	
 
 	[CmdletBinding( SupportsShouldProcess=$true )]
 	param(
 		[Parameter(Mandatory=$true,ValueFromPipeline=$true)]
 		[Alias("Name")]
 		[string]
 		$targetName,	
 
 		[Parameter()]
 		[string]
 
 specifying * for this parameter, the iSCSI initiator service will use the initiator node name as the CHAP username.
 		$username,
 
 		[Parameter()]
 		[string]
 
 initiator will use this secret to compute a hash value based on the challenge sent by the target. 
 		$password,
 		
 		[Parameter()]
 		[switch]
 		$persist
 	)
 	process
 	{	
 		if( $username )
 		{
 			$data = & iscsicli qlogintarget $targetName $username $password
 			Write-Verbose "Raw iSCSIcli output: $data";
 		}
 		else
 		{
 			$data = & iscsicli qlogintarget $targetName
 			Write-Verbose "Raw iSCSIcli output: $data";
 			$username = '*';
 			$password = '*';
 
 		}
 
 		Write-Verbose "Raw iSCSIcli output: $data";
 		
 		if( $data -match 'already.+logged' )
 		{
 			$s = get-session | where { $_.targetname -eq $targetName };
 			New-Object psobject -Property @{ SessionId=$s.SessionId; ConnectionId=$s.Connection.ConnectionID 
 
 };
 		}
 		else
 		{
 		
 			( $data | Out-String ) -replace '0x','' -replace "[`r`n]+",'=' | convertFrom-iSCSIOutput -field ' 
 
 is ';
 		}
 
 		if( $persist )
 		{
 			& iscsicli persistentlogintarget $targetName T * * * * * * * * * * * $username $password * * 0 | 
 
 Out-Null
 		}
 	}
 }
 
 #
 function Disconnect-Session
 {
 <#
 This command will attempt to logout of a target which was logged in via the session specified by SessionId. The iSCSI 
 
 initiator service will not logout of a session if any devices exposed by it are currently in use.  If the command fails 
 
 then consult the system eventlog for additional information about the component that is using the device.
 #>
 	[CmdletBinding( SupportsShouldProcess=$true, ConfirmImpact='high' )]
 	param(
 		[Parameter( Mandatory=$true, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true )]
 		[string]
 		$sessionId,
 		
 		[Parameter()]
 		[switch]
 		$force
 	)
 	process
 	{
 		if( -not( $force -or $pscmdlet.shouldProcess( $sessionId, "Disconnect Session" ) ) )
 		{
 			return;
 		}
 		
 		$data = & iscsicli logouttarget $sessionId | Out-String;				
 		if( $data -notmatch 'success' )
 		{
 			throw $data;
 		}
 	}
 	
 }
 
 
 #
 function Get-Initiators
 {
 <#
 This command will display the list of initiator instance names that are running and operating with the iSCSI initiator 
 
 service.
 #>
 	[CmdletBinding()]
 	param()
 	process
 	{
 		& iscsicli listinitiators
 	}
 }
 
 #
 function Get-Session
 {
 <#
 This command displays the list of active sessions for all initiators. Note that a session that has no connections is not 
 
 connected to the target and is in a retry state.
 
 Microsoft iSCSI Initiator Version 6.1 Build 7601
 
 Total of 2 sessions
 
 Session Id             : fffffa800f7900a8-400001370000000d
 Initiator Node Name    : iqn.1991-05.com.microsoft:archimedes
 Target Node Name       : (null)
 Target Name            : iqn.2008-08.com.starwindsoftware:127.0.0.1-target1
 ISID                   : 40 00 01 37 00 00
 TSID                   : 27 00
 Number Connections     : 1
 
     Connections:
 
         Connection Id     : fffffa800f7900a8-1b
         Initiator Portal  : 0.0.0.0/58847
         Target Portal     : 192.168.1.108/3260
         CID               : 01 00
 
     Devices:
         Device Type            : Disk
         Device Number          : 1
         Storage Device Type    : 7
         Partition Number       : 0
         Friendly Name          : ROCKET RAM DISK 1024 M SCSI Disk Device
         Device Description     : Disk drive
         Reported Mappings      : Port 1, Bus 0, Target Id 0, LUN 0
         Location               : Bus Number 0, Target Id 0, LUN 0
         Initiator Name         : ROOT\ISCSIPRT\0000_0
         Target Name            : iqn.2008-08.com.starwindsoftware:127.0.0.1-target1
 4f2-00a0c91efb8b}
         Legacy Device Name     : \\.\PhysicalDrive1
         Device Instance        : 0x82c
         Volume Path Names      :
                                  E:\
 
 Session Id             : fffffa800f7900a8-400001370000000f
 Initiator Node Name    : iqn.1991-05.com.microsoft:archimedes
 Target Node Name       : (null)
 Target Name            : iqn.2008-08.com.starwindsoftware:127.0.0.1-scratch
 ISID                   : 40 00 01 37 00 00
 TSID                   : 2b 00
 Number Connections     : 1
 
     Connections:
 
         Connection Id     : fffffa800f7900a8-1d
         Initiator Portal  : 0.0.0.0/59359
         Target Portal     : 192.168.1.106/3260
         CID               : 01 00
 
     Devices:
         Device Type            : Disk
         Device Number          : 2
         Storage Device Type    : 7
         Partition Number       : 0
         Friendly Name          : ROCKET RAM DISK 256 MB SCSI Disk Device
         Device Description     : Disk drive
         Reported Mappings      : Port 1, Bus 0, Target Id 1, LUN 0
         Location               : Bus Number 0, Target Id 1, LUN 0
         Initiator Name         : ROOT\ISCSIPRT\0000_0
         Target Name            : iqn.2008-08.com.starwindsoftware:127.0.0.1-scratch
 4f2-00a0c91efb8b}
         Legacy Device Name     : \\.\PhysicalDrive2
         Device Instance        : 0x8ac
 
 #>
 	[CmdletBinding()]
 	param(
 		[Parameter( ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true )]
 		[string]
 		$sessionId = '.*'
 	)
 	process
 	{
 		Write-Verbose "Session ID filter: $sessionId";
 		$data = ( & iscsicli sessionlist ) | out-string;
 		$data = $data -replace "[`r`n]+",'='
 		Write-Verbose "raw sessionlist info : $data";
 		
 		$sessions = $data -split "Session Id\s+:\s+";
 		$sessions = $sessions | where { $_ -match "Connections:" } | foreach {
 			$session, $data = ( "Session Id : " + $_ ) -split 'Connections:', 2;
 			$connection, $device = $data -split "Devices:", 2;
 			
 			Write-Verbose "session $session";
 			Write-Verbose "connection $connection";
 			Write-Verbose "device $device";	
 			
 			$session, $connection, $device = $session, $connection, $device | convertFrom-iSCSIOutput;
 			$session | Add-Member -PassThru -MemberType NoteProperty -Name Connection -Value $connection |
 					Add-Member -MemberType NoteProperty -Name Device -Value $device;
 			$session;
 		}
 		
 		if( -not $sessions )
 		{
 			Write-Verbose "no sessions found"
 			return;
 		}
 		
 		$sessions | write-verbose;		
 		$sessions | where { write-verbose "filtering $($_.SessionId) by $sessionId"; $_.SessionId -match 
 
 $sessionId }
 	}
 }
 
 function convertFrom-iSCSIOutput
 {
 	[CmdletBinding()]
 	param(
 		[Parameter(ValueFromPipeline=$true)]
 		[string]
 		$data,
 		
 		[Parameter()]
 		[string]
 		$itemDelimiter = '=',
 		
 		[Parameter()]
 		[string]
 		$fieldSeparator = ':'
 	)
 	
 	process
 	{
 		Write-Debug "convertFrom-iSCSIOutput ..."
 		Write-Debug "  Data: $data";
 		Write-Debug "  Item Delimiter: $itemdelimiter";
 		Write-Debug "  Field Separator: $fieldSeparator";
 		$a = @{};
 		
 		$data -split $itemDelimiter | where { $_ -match "$fieldSeparator\s*" } | foreach {			
 					
 			function add-ToA( $k, $v )
 			{
 				$k = $k -replace ' ','';
 				Write-Debug "item key $k; value $v";
 			
 				$a[$k] = $v;
 			}
 
 			Write-Debug "item entry $_";
 			
 			$k,$v = $_ -split "\s*$fieldSeparator\s*",2;
 			if( $k -match ' and ' )
 			{
 				$k1, $k2 = $k -split ' and ';
 				$v1, $v2 = $v -split '\s+',2;
 				add-ToA $k1 $v1	
 				add-ToA $k2 $v2
 			}
 			else
 			{
 				add-ToA $k $v
 			}
 		}		
 		new-object psobject -Property $a;		
 	}
 }
 
 ###########################################################
 
 Export-ModuleMember -Function '*';
`

