---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 4.7
Encoding: ascii
License: cc0
PoshCode ID: 5374
Published Date: 2014-08-19t23
Archived Date: 2014-08-24t13
---

# sharppcap - 

## Description

your first steps at capturing packets from powershell with winpcap via sharppcap

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 Add-Type -Path $SkyProfileDir\Libraries\SharpPcap*\SharpPcap.dll
 
 Update-TypeData -TypeName SharpPcap.WinPcap.WinPcapDevice `
     -MemberName FriendlyName -MemberType ScriptProperty -Value { $this.Interface.FriendlyName }
 Update-TypeData -TypeName SharpPcap.WinPcap.WinPcapDevice `
     -DefaultDisplayProperty FriendlyName -DefaultDisplayPropertySet FriendlyName, Addresses, Description, MacAddress
 Update-TypeData -TypeName SharpPcap.WinPcap.WinPcapDevice `
     -MemberName MacAddress -MemberType ScriptProperty -Value { $this.Interface.MacAddress } -Force
 Update-TypeData -TypeName SharpPcap.LibPcap.PcapAddress `
     -DefaultDisplayProperty Addr -MemberName ToString -MemberType ScriptMethod -Value { $this.Addr.ToString() } -Force
 
 [SharpPcap.CaptureDeviceList]::Instance | Format-Table FriendlyName, Addresses, Description, MacAddress -Auto -Wrap
 
 
 
 Import-Module Reflection -Min 4.7 -ErrorAction SilentlyContinue
 if($choose = get-command Read-Choice* | ? { $_.Parameters.Values.GetEnumerator() | % Attributes | % ValueFromPipeline }) {
     $device = [SharpPcap.CaptureDeviceList]::Instance |
        &$choose -Prompt "Please choose a device for capture:" -Label FriendlyName -Passthru
 } else {
     $device = [SharpPcap.CaptureDeviceList]::Instance | 
               Where { $_.Addresses.Addr -like "192.*" } | 
               Select -First 1
 }
 
 Get-EventSubscriber -Source $device.MacAddress -EA 0 | Where EventName -eq OnPacketArrival | Unregister-Event
 $null = Register-ObjectEvent $device -EventName OnPacketArrival -Source $device.MacAddress {
     try {
         $Packet = [PacketDotNet.Packet]::ParsePacket($EventArgs.Packet.LinkLayerType, $EventArgs.Packet.Data) 
 
 
         Write-Host ([DateTime]$EventArgs.Packet.Timeval.Date) "Len=" $EventArgs.Packet.Data.Length -Foreground Cyan
         Write-Host $Packet.ToString()
     } catch {
         Write-Host "ERROR:" $_ -Foreground Red
     }
 }
 
 
 Write-Host "Capturing on" $device.FriendlyName
 
 $device.Open([SharpPcap.DeviceMode]::Promiscuous, 1000);
 $device.Filter = "tcp"
 $device.StartCapture()
 
 for($msec=0; $msec -lt 3000; $msec += 100) {
     Start-Sleep -milli 100
     Write-Host -NoNewLine
 }
 
 $device.StopCapture()
 $device.Close()
`

