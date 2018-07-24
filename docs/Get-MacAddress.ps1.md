---
Author: terran
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6458
Published Date: 2016-07-27t13
Archived Date: 2016-09-07t05
---

# get-macaddress - 

## Description

found at https

## Comments



## Usage



## TODO



## function

`get-macaddress`

## Code

`#
 #
 function Get-MacAddress {
   <#
     .SYNOPSIS
         Gets MAC address.
     .NOTES
         Alternative ways to do same.
         
         Example 1:
         $asm = Add-Type -MemberDefinition @'
           [DllImport("rpcrt4.dll")]
           public static extern Int32 UuidCreateSequential(
               out Guid guid
           );
         '@ -Name Uuid -NameSpace MacAddress -PassThru
         $guid = New-Object Guid
         
         if (($res = $asm::UuidCreateSequential([ref]$guid)) -ne 0) {
           (New-Object ComponentModel.Win32Exception($res)).Message
           break
         }
         [Regex]::Replace(
           $guid.Guid.Split('-')[-1], '.{2}', '$0-'
         ).TrimEnd('-')
         
         Example 2:
         Get-WmiObject Win32_NetworkAdapter | Where-Object {
           $_.MacAddress
         } | ForEach-Object {
           New-Object PSObject -Property @{
             Description = $_.Description
             Service     = $_.ServiceName
             MACAddress  = $_.MACAddress
           }
         } | Select-Object Description, Service, MACAddress
         
         Example 3:
         Get-WmiObject Win32_NetworkAdapterConfiguration | Where-Object {
           $_.MacAddress
         } | ForEach-Object {
           New-Object PSObject -Property @{
             Description = $_.Description
             Id          = $_.SettingID
             MACAddress  = $_.MACAddress
           }
         } | Select-Object Description, Id, MACAddress
         
         Example 4:
         $mac, $nic = (getmac /fo csv | Where-Object {
           ![String]::IsNullOrEmpty($_) -and $_ -match '\w{2}\-'
         }).Split(',') | ForEach-Object {$_.Trim('"')}
         New-Object PSObject -Property @{
           Interface  = $nic.Substring(($$ = $nic.IndexOf('{')), $nic.Length - $$)
           MACAddress = $mac
         }
         
         Example 5:
         $key = 'HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\*'
         Get-ItemProperty $key | Where-Object {
           $_.DhcpIpAddress -and $_.DhcpIpAddress -ne '0.0.0.0'
         } | Select-Object DhcpIpAddress | ForEach-Object {
           New-Object PSObject -Property @{
             IPAddress  = $_.DhcpIpAddress
             MACAddress = ([Regex]'(\w{2}\-){5}\w{2}').Match((
               nbtstat -a $_.DhcpIpAddress
             )).Value
           }
         }
         
         Example 6:
         ([Regex]'(\w{2}\-){5}\w{2}').Match((
             ipconfig /all
         )).Value
         
         Example 7:
         ([Regex]'(\w{2}\s){6}').Match((
             route print
         )).Value.Trim().Replace([Char]32, '-')
   #>
   
   if (($clr = $PSVersionTable.CLRVersion.Major) -ge 4) {
     [Net.NetworkInformation.NetworkInterface]::GetAllNetworkInterfaces() |
     Where-Object {
       $_.OperationalStatus -eq [Net.NetworkInformation.OperationalStatus]::Up
     } | ForEach-Object {
       if (![String]::IsNullOrEmpty((
         $$ = $_.GetPhysicalAddress().ToString()
       ))) {
         New-Object PSObject -Property @{
           Id         = $_.Id
           MACAddress = [Regex]::Replace($$, '.{2}', '$0-').TrimEnd('-')
         }
       }
     } | Select-Object Description, Id, MACAddress | Format-List
   }
   elseif ($clr -eq 2) {
     @(
       [Runtime.InteropServices.CallingConvention],
       [Runtime.InteropServices.Marshal],
       [Reflection.BindingFlags],
       [Reflection.Emit.OpCodes]
     ) | ForEach-Object {
       $keys = ($ta = [PSObject].Assembly.GetType(
         'System.Management.Automation.TypeAccelerators'
       ))::Get.Keys
       $collect = @()
     }{
       if ($keys -notcontains $_.Name) {
         $ta::Add($_.Name, $_)
       }
       $collect += $_.Name
     }
     
     function Get-LastError {
       param(
         [Int32]$ErrorCode = [Marshal]::GetLastWin32Error()
       )
       
       [PSObject].Assembly.GetType(
         'Microsoft.PowerShell.Commands.Internal.Win32Native'
       ).GetMethod(
         'GetMessage', [BindingFlags]40
       ).Invoke(
         $null, @($ErrorCode)
       )
     }
     
     function private:Invoke-FreeLibrary {
       param(
         [Parameter(Mandatory=$true)]
         [IntPtr]$ModuleHandle
       )
       
       [void][Linq.Enumerable].Assembly.GetType(
         'Microsoft.Win32.UnsafeNativeMethods'
       ).GetMethod(
         'FreeLibrary', [BindingFlags]40
       ).Invoke($null, @($ModuleHandle))
     }
     
     function private:Get-ProcAddress {
       param(
         [Parameter(Mandatory=$true, Position=0)]
         [ValidateNotNullOrEmpty()]
         [String]$Module,
         
         [Parameter(Mandatory=$true, Position=1)]
         [ValidateNotNullOrEmpty()]
         [String]$Function
       )
       
       [Data.Rule].Assembly.GetType(
         'System.Data.Common.SafeNativeMethods'
       ).GetMethods(
         [BindingFlags]40
       ) | Where-Object {
         $_.Name -cmatch '\AGet(ProcA|ModuleH)'
       } | ForEach-Object {
         Set-Variable $_.Name $_
       }
       
       if (($ptr = $GetModuleHandle.Invoke(
         $null, @($Module)
       )) -eq [IntPtr]::Zero) {
         if (($mod = [Regex].Assembly.GetType(
           'Microsoft.Win32.SafeNativeMethods'
         ).GetMethod('LoadLibrary').Invoke(
           $null, @($Module)
         )) -eq [IntPtr]::Zero) {
           Write-Warning "$(Get-LastError)"
           break
         }
         $ptr = $GetModuleHandle.Invoke($null, @($Module))
       }
       
       $GetProcAddress.Invoke($null, @($ptr, $Function)), $mod
     }
     
     function private:Set-Delegate {
       param(
         [Parameter(Mandatory=$true, Position=0)]
         [ValidateScript({$_ -ne [IntPtr]::Zero})]
         [IntPtr]$ProcAddress,
         
         [Parameter(Mandatory=$true, Position=1)]
         [ValidateNotNullOrEmpty()]
         [String]$Delegate
       )
       
       $proto = Invoke-Expression $Delegate
       $method = $proto.GetMethod('Invoke')
       
       $returntype = $method.ReturnType
       $paramtypes = $method.GetParameters() |
                                     Select-Object -ExpandProperty ParameterType
       
       $holder = New-Object Reflection.Emit.DynamicMethod(
         'Invoke', $returntype, $paramtypes, $proto
       )
       $il = $holder.GetILGenerator()
       0..($paramtypes.Length - 1) | ForEach-Object {
         $il.Emit([OpCodes]::Ldarg, $_)
       }
       
       switch ([IntPtr]::Size) {
         4 { $il.Emit([OpCodes]::Ldc_I4, $ProcAddress.ToInt32()) }
         8 { $il.Emit([OpCodes]::Ldc_I8, $ProcAddress.ToInt64()) }
       }
       $il.EmitCalli(
         [OpCodes]::Calli, [CallingConvention]::StdCall, $returntype, $paramtypes
       )
       $il.Emit([OpCodes]::Ret)
       
       $holder.CreateDelegate($proto)
     }
     
     $ptr, $mod = Get-ProcAddress iphlpapi SendARP
     $SendARP = Set-Delegate $ptr `
                               '[Func[UInt32, UInt32, [Byte[]], [Byte[]], Int32]]'
     
     $key = 'HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces\*'
     Get-ItemProperty $key | Where-Object {
       $_.DhcpIpAddress -and $_.DhcpIpAddress -ne '0.0.0.0'
     } | ForEach-Object {
       $inet_addr = [Regex].Assembly.GetType(
         'System.Net.UnsafeNclNativeMethods+OSSOCK'
       ).GetMethod('inet_addr', [BindingFlags]40)
     }{
       $adr = [BitConverter]::ToUInt32(
         [BitConverter]::GetBytes(
           $inet_addr.Invoke($null, @($_.DhcpIpAddress)
         )), 0
       )
       $mac = New-Object Byte[] 6
       $len = [BitConverter]::GetBytes($mac.Length)
       
       if (($ret = $SendARP.Invoke($adr, 0, $mac, $len)) -ne 0) {
         Write-Warning "$(Get-LastError $ret)"
         break
       }
       
       New-Object PSObject -Property @{
         Id         = $_.PSChildName
         MACAddress = ($mac | ForEach-Object {'{0:X2}' -f $_}) -join '-'
       }
     }
     
     if ($mod) { Invoke-FreeLibrary $mod }
     $collect | ForEach-Object { [void]$ta::Remove($_) }
   }
 }
`

