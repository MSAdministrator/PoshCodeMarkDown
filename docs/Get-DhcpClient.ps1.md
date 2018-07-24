---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2310
Published Date: 
Archived Date: 2010-10-21t23
---

# get-dhcpclient - 

## Description

more c# than powershell. a script to return dhcp client leases from a dhcp server.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
   .Description
   .Parameter Server
     A DHCP server to query
   .Parameter Scope
     The IP address representing the scope to query
 #>
 
 [CmdLetBinding()]
 Param(
   [Parameter(Mandatory = $True)]
   [Net.IPAddress]$Server,
   [Parameter(Mandatory = $True)]
   [Net.IPAddress]$Scope
 )
 
 #
 #
 
 Try { [Void][Dhcp.DhcpClient] } Catch { }
 If (!$?) {
 
 Add-Type @"
   using System;
   using System.Collections;
   using System.Net;
   using System.Runtime.InteropServices;
   using Dhcp;
 
   namespace Dhcp
   {
     public class ConvertTo {
 
       public static IPAddress IP(UInt32 Value)
       {
         Byte[] IPArray = new Byte[4];
 
         for (int i = 3; i > -1; i--) {
           Double Remainder = Value % Math.Pow(256, i);
           IPArray[3 - i] = (Byte)((Value - Remainder) / Math.Pow(256, i));
           Value = (UInt32)Remainder;
         }
 
         return IPAddress.Parse(String.Format("{0}.{1}.{2}.{3}", 
           IPArray[0],
           IPArray[1],
           IPArray[2],
           IPArray[3]));
       }
 
       public static UInt32 UInt32(IPAddress IP)
       {
         UInt32 Value = 0;
         Byte[] Bytes = IP.GetAddressBytes();
         for (int i = 0; i < 4; i++) {
           Value = Value | (UInt32)(Bytes[i] << (8 * (3 - i)));
         }
         return Value;
       }
     }
 
     public class Functions {
 
       [DllImport("dhcpsapi.dll")]
       public static extern UInt32 DhcpEnumSubnetClients(
         [MarshalAs(UnmanagedType.LPWStr)]
         String ServerIpAddress,
         uint SubnetAddress,
         ref uint ResumeHandle,
         uint PreferredMaximum,
         out IntPtr ClientInfo,
         out uint ClientsRead,
         out uint ClientsTotal
       );
 
     }
 
     public class Structures {
 
       [StructLayout(LayoutKind.Sequential)]
       public struct DATE_TIME
       {
         public UInt32 dwLowDateTime;
         public UInt32 dwHighDateTime;
 
         public DateTime ToDateTime()
         {
           if (dwHighDateTime == 0 && dwLowDateTime == 0)
           {
             return DateTime.MinValue;
           }
           if (dwHighDateTime == int.MaxValue && dwLowDateTime == UInt32.MaxValue)
           {
             return DateTime.MaxValue;
           }
           return DateTime.FromFileTime((((long)dwHighDateTime) << 32) | dwLowDateTime);
         }
       }
 
       [StructLayout(LayoutKind.Sequential)]
       public struct DHCP_BINARY_DATA
       {
         public uint DataLength;
         public IntPtr Data;
 
         public override String ToString()
         {
           return String.Format("{0:X2}:{1:X2}:{2:X2}:{3:X2}:{4:X2}:{5:X2}",
             Marshal.ReadByte(this.Data),
             Marshal.ReadByte(this.Data, 1),
             Marshal.ReadByte(this.Data, 2),
             Marshal.ReadByte(this.Data, 3),
             Marshal.ReadByte(this.Data, 4),
             Marshal.ReadByte(this.Data, 5));
         }
       };
 
       [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
       public struct DHCP_CLIENT_INFO_ARRAY
       {
         public uint NumElements;
         public IntPtr Clients;
       }
 
       [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Auto)]
       public struct DHCP_CLIENT_INFO
       {
         public uint ClientIpAddress;
         public uint SubnetMask;
         public DHCP_BINARY_DATA ClientHardwareAddress;
         [MarshalAs(UnmanagedType.LPWStr)]
         public string ClientName;
         [MarshalAs(UnmanagedType.LPWStr)]
         public string ClientComment;
         public DATE_TIME ClientLeaseExpires;
         public DHCP_HOST_INFO OwnerHost;
       }
 
       [StructLayout(LayoutKind.Sequential)]
       public struct DHCP_HOST_INFO
       {
         uint IpAddress;
         [MarshalAs(UnmanagedType.LPWStr)]
         String NetBiosName;
         [MarshalAs(UnmanagedType.LPWStr)]
         String HostName;
       }
     }
 
     public class DhcpClient
     {
       public String Name;
       public String MACAddress;
       public IPAddress IpAddress;
       public IPAddress SubnetMask;
       public String Description;
       public DateTime LeaseExpires;
 
       internal DhcpClient(Structures.DHCP_CLIENT_INFO RawReservation)
       {
         this.IpAddress = ConvertTo.IP(RawReservation.ClientIpAddress);
         this.SubnetMask = ConvertTo.IP(RawReservation.SubnetMask);
         this.MACAddress = RawReservation.ClientHardwareAddress.ToString();
         this.Name = RawReservation.ClientName;
         this.Description = RawReservation.ClientComment;
         this.LeaseExpires = RawReservation.ClientLeaseExpires.ToDateTime();
       }
 
       public static DhcpClient[] Get(IPAddress ServerIP, IPAddress ScopeIP) {
         ArrayList Clients = new ArrayList();
         uint resumeHandle = 0;
         IntPtr info_array_ptr;
         uint numClientsRead = 0;
         uint totalClients = 0;
 
         String Server = ServerIP.ToString();
         UInt32 Scope = ConvertTo.UInt32(ScopeIP);
 
         UInt32 ReturnCode = Functions.DhcpEnumSubnetClients(  
           Server,
           Scope,
           ref resumeHandle,
           65536,
           out info_array_ptr,
           out numClientsRead,
           out totalClients
         );
 
         Structures.DHCP_CLIENT_INFO_ARRAY rawClients =  
           (Structures.DHCP_CLIENT_INFO_ARRAY)Marshal.PtrToStructure(info_array_ptr, typeof(Structures.DHCP_CLIENT_INFO_ARRAY));
         IntPtr current = rawClients.Clients;
 
         for (int i = 0; i < (int)rawClients.NumElements; i++) {
           Structures.DHCP_CLIENT_INFO rawMachine =  
             (Structures.DHCP_CLIENT_INFO)Marshal.PtrToStructure(Marshal.ReadIntPtr(current), typeof(Structures.DHCP_CLIENT_INFO));
 
           Clients.Add(new DhcpClient(rawMachine));
 
           current = (IntPtr)((int)current + (int)Marshal.SizeOf(typeof(IntPtr)));  
         }
 
         return (DhcpClient[])Clients.ToArray(typeof(DhcpClient));
       }
     }
   }
 "@
 
 }
 
 #
 #
 
 [Dhcp.DhcpClient]::Get($Server, $Scope)
`

