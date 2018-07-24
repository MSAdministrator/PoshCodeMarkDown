---
Author: christian bellee
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1477
Published Date: 2010-11-19t19
Archived Date: 2017-04-14t01
---

# list dhcp clients - 

## Description

function to list all clients in a microsoft dhcp server database. the script uses p/invoke function signatures, structs & ports the dhcpenumsubnetclients() c# code example to powershell from pinvoke.net. there is no other (nice!) way to get this information in an object form, other than scraping the output from netsh.exe.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 $DHCP_EnumSubnetClients = @'          
     [DllImport("dhcpsapi.dll", SetLastError = true, CharSet = CharSet.Auto)]
 public static extern uint DhcpEnumSubnetClients(
     string ServerIpAddress,
     uint SubnetAddress,
     ref uint ResumeHandle,
     uint PreferredMaximum,
     out IntPtr ClientInfo,
     ref uint ElementsRead,
     ref uint ElementsTotal);
 '@
 
 $DHCP_Structs = @'
 namespace mystruct {
 using System;
 using System.Runtime.InteropServices;
 
     public struct CUSTOM_CLIENT_INFO
     {
     public string ClientName;
     public string IpAddress;
     public string MacAddress;
     }
 
     [StructLayout(LayoutKind.Sequential)]
     public struct DHCP_CLIENT_INFO_ARRAY
     {
     public uint NumElements;
     public IntPtr Clients;
     }
 
     [StructLayout(LayoutKind.Sequential)]
     public struct DHCP_CLIENT_UID
     {
     public uint DataLength;
     public IntPtr Data;
     }
 
     [StructLayout(LayoutKind.Sequential)]
     public struct DATE_TIME
     {
     public uint dwLowDateTime;
     public uint dwHighDateTime;
     
     public DateTime Convert()
     {
         if (dwHighDateTime== 0 && dwLowDateTime == 0)
         {
         return DateTime.MinValue;
         }
         if (dwHighDateTime == int.MaxValue && dwLowDateTime == UInt32.MaxValue)
         {
         return DateTime.MaxValue;
         }
         return DateTime.FromFileTime((((long) dwHighDateTime) << 32) | (UInt32) dwLowDateTime);
         }
     }
     
     [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
     public struct DHCP_HOST_INFO
     {
     public uint IpAddress;
     public string NetBiosName;
     public string HostName;
     }
 
     [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
     public struct DHCP_CLIENT_INFO
     {
     public uint ClientIpAddress;
     public uint SubnetMask;
     public DHCP_CLIENT_UID ClientHardwareAddress;
     [MarshalAs(UnmanagedType.LPWStr)]
     public string ClientName;
     [MarshalAs(UnmanagedType.LPWStr)]
     public string ClientComment;
     public DATE_TIME ClientLeaseExpires; 
     public DHCP_HOST_INFO OwnerHost; 
     }
 }
 '@
 
 $resumeHandle = 0
 $clientInfo = 0
 $ElementsRead = 0
 $ElementsTotal = 0
 
 Add-Type $DHCP_Structs 
 Add-Type  -MemberDefinition $DHCP_EnumSubnetClients -Name GetDHCPInfo -Namespace Win32DHCP
 
 
 [void][Win32DHCP.GetDHCPInfo]::DhcpEnumSubnetClients($DHCPServerIP,0,[ref]$resumeHandle,0,[ref]$clientInfo,[ref]$ElementsRead,[ref]$ElementsTotal)
 
 $clients = [system.runtime.interopservices.marshal]::PtrToStructure($clientInfo,[mystruct.DHCP_CLIENT_INFO_ARRAY])
 
 [int]$size = $clients.NumElements
 [int]$current = $clients.Clients
 $ptr_array = new-object system.intptr[]($size)
 $current = new-object system.intptr($current)
 
 for ($i=0;$i -lt $size;$i++) 
 	{
 	$ptr_array[$i] = [system.runtime.interopservices.marshal]::ReadIntPtr($current)
 	$current = $current + [system.runtime.interopservices.marshal]::SizeOf([system.IntPtr])
 	}
 
 function uIntToIP {
 param ($intIP)
 $objIP = new-object system.net.ipaddress($intIP)
 $arrIP = $objIP.IPAddressToString.split(".")
 return $arrIP[3] + "." + $arrIP[2] + "." + $arrIP[1] + "." + $arrIP[0]
 }
 
 [array]$clients_array = new-object mystruct.CUSTOM_CLIENT_INFO
 
 for ($i=0;$i -lt $size;$i++) 
 	{
 	$objDHCPInfo = New-Object psobject
 	$current_element = [system.runtime.interopservices.marshal]::PtrToStructure($ptr_array[$i],[mystruct.DHCP_CLIENT_INFO])
 	add-member -inputobject $objDHCPInfo -memberType noteproperty -name ClientIP -value $(uIntToIP $current_element.ClientIpAddress)
 	add-member -inputobject $objDHCPInfo -memberType noteproperty -name ClientName -value $current_element.ClientName
 	add-member -inputobject $objDHCPInfo -memberType noteproperty -name OwnerIP -value $(uIntToIP $current_element.Ownerhost.IpAddress)
 	add-member -inputobject $objDHCPInfo -memberType noteproperty -name OwnerName -value $current_element.Ownerhost.NetBiosName
 	add-member -inputobject $objDHCPInfo -memberType noteproperty -name SubnetMask -value $(uIntToIP $current_element.SubnetMask)
 	add-member -inputobject $objDHCPInfo -memberType noteproperty -name LeaseExpires -value $current_element.ClientLeaseExpires.Convert()
  
 	$mac = [System.String]::Format(
 	"{0:x2}-{1:x2}-{2:x2}-{3:x2}-{4:x2}-{5:x2}",
 	[system.runtime.interopservices.marshal]::ReadByte($current_element.ClientHardwareAddress.Data),
 	[system.runtime.interopservices.marshal]::ReadByte($current_element.ClientHardwareAddress.Data, 1),
 	[system.runtime.interopservices.marshal]::ReadByte($current_element.ClientHardwareAddress.Data, 2),
 	[system.runtime.interopservices.marshal]::ReadByte($current_element.ClientHardwareAddress.Data, 3),
 	[system.runtime.interopservices.marshal]::ReadByte($current_element.ClientHardwareAddress.Data, 4),
 	[system.runtime.interopservices.marshal]::ReadByte($current_element.ClientHardwareAddress.Data, 5)
 	)
 
 add-member -inputobject $objDHCPInfo -memberType noteproperty -name MacAddress -value $mac
 $objDHCPInfo
 }
`

