---
Author: glnsize
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1895
Published Date: 2011-06-03t12
Archived Date: 2016-03-18t21
---

# dvswitchnetworkadapter - 

## Description

a simple module to temporarily replace the stock powercli networkadapter cmdlets until they work with vdswitches. based off lucd’s functions http

## Comments



## Usage



## TODO



## script

`get-dvswitchnetworkadapter`

## Code

`#
 #
 <#
     .SYNOPSIS
         Retrieves the virtual network adapters  available on a vSphere server.
     .DESCRIPTION
         A replacement for the default cmdlet will work with either a standard vSwitch or a dvSwitch. 
         Retrieves the virtual network adapters  available on a vSphere server.
     .PARAMETER VM
         Specify the virtual machines from which you want to retrieve virtual network adapters.
     .EXAMPLE
         Get-VM | Get-dvSwitchNetworkAdapter
     .OUTPUTS
         PSObject
     .LINK
         Set-dvSwitchNetworkAdapter
 #>
 Function Get-dvSwitchNetworkAdapter
 {
     [cmdletbinding()]
     param(
         [parameter(Mandatory = $true, ValueFromPipeline = $true)]
         [VMware.VimAutomation.Client20.VirtualMachineImpl[]]
         $VM
     )
     process 
     {
         Foreach ($VirtualMachine in (get-view -VIObject $VM -Property "Config.Hardware.Device","Network")) 
         {
             $VirtualVmxnets = $VirtualMachine.Config.Hardware.Device |
                 Where-Object {"VirtualEthernetCard","VirtualVmxnet" -contains $_.GetType().BaseType.Name}
             Foreach ($VirtualVmxnet in $VirtualVmxnets) 
             {
                 New-Object PSObject -Property @{
                     MacAddress = $VirtualVmxnet.MacAddress
                     WakeOnLanEnabled = $VirtualVmxnet.WakeOnLanEnabled
                     NetworkName = switch ($VirtualVmxnet.Backing.GetType().Name) 
                         {
                             "VirtualEthernetCardNetworkBackingInfo" 
                             {
                                 Get-View $VirtualVmxnet.Backing.network -Property 'Name'|
                                     Select-Object -ExpandProperty 'Name'
                             }
                             Default 
                             {
                               Get-view -ID $VirtualMachine.Network -Property 'Name','Key' |
                                 Where-Object {$_.Key -eq $VirtualVmxnet.Backing.Port.PortgroupKey} |
                                 Select-Object -ExpandProperty 'Name'
                             }
                         }
                     Type = $VirtualVmxnet.GetType().Name
                     ParentId = $VirtualMachine.MoRef.Type + "-" + $VirtualMachine.MoRef.Value
                     ConnectionState = New-Object PSObject -Property @{
                         Connected = $VirtualVmxnet.Connectable.Connected
                         StartConnected = $VirtualVmxnet.Connectable.StartConnected
                     }
                     Id = "{0}-{1}/{2}" -f $VirtualMachine.MoRef.Type,$VirtualMachine.MoRef.Value, $VirtualVmxnet.Key
                     Name = $VirtualVmxnet.DeviceInfo.Label
                 }
             }
         }
     }
 }
 
 <#
     .SYNOPSIS
          Changes the configuration of the virtual network adapter.
     .DESCRIPTION
         A replacement for the default cmdlet will work with either a standard vSwitch or a dvSwitch. 
         Changes the configuration of the virtual network adapter.
     .PARAMETER NetworkAdapter
         Specify the name of the network to which you want to connect the virtual network adapter.
         NetworkName must be globally unique within vSphere.
     .PARAMETER NetworkName
         Specify the virtual network adapter you want to configure.
     .PARAMETER StartConnected
         If the value is $true, the virtual network adapter starts connected when its associated 
         virtual machine powers on. If the value is $false, it starts disconnected.  If Nothing is
         Supplied than nothing will be changed.
     .PARAMETER Connected
         If the value is $true, the virtual network adapter is connected after its creation. If the 
         value is $false, it is disconnected.  If Nothing is Supplied than nothing will be changed.
     .RunAsync
         If supplied then the cmdlet will not wait for the completion of the network reconfiguration.  
         Instead a Task object will be returned and the cmdlet will process the next object in the
         pipeline.
     .EXAMPLE
         Get-VM | Get-dvSwitchNetworkAdapter | Where-object{$_.NetworkName -eq "192.168.1.0"}| Set-dvSwitchNetworkAdapter -NetworkName "dvSwitch0_192.168.1.0"
     .EXAMPLE
         Get-VM | Get-dvSwitchNetworkAdapter | Set-dvSwitchNetworkAdapter -StartConnected $True
     .OUTPUTS
         PSObject
     .LINK
         Get-dvSwitchNetworkAdapter
 #>
 Function Set-dvSwitchNetworkAdapter
 {
     [cmdletbinding(SupportsShouldProcess=$true)]
     param(
         [parameter(Mandatory = $true, ValueFromPipeline = $true)]
         [PSCustomObject[]]
         $NetworkAdapter
     ,
         [parameter(Mandatory = $false)]
         [string]
         $NetworkName
     ,
         [parameter()]
         [bool]
         $StartConnected
     ,
         [parameter()]
         [bool]
         $Connected
     ,
         [parameter()]
         [switch]
         $RunAsync
     )
     process 
     {
         Foreach ($netAdapter in $NetworkAdapter) 
         {
             $VirtualMachine = Get-View $netAdapter.ParentId -Property 'Runtime.Host','Config.Hardware.Device','Name'
             $HostSystem = Get-View -ID $VirtualMachine.Runtime.Host -Property 'Network','Name'
            
 
             $VirtualDeviceConfigSpec = New-Object VMware.Vim.VirtualDeviceConfigSpec
             $VirtualDeviceConfigSpec.operation = "edit"
             Try 
             {
                 $VirtualDeviceConfigSpec.device = $VirtualMachine.Config.Hardware.Device |
                     Where-Object {$_.GetType().Name -eq $netAdapter.Type -and $_.MacAddress -eq $netAdapter.MacAddress}
             }
             Catch
             {
                 write-warning "$($netAdapter.Name) not found on $($VirtualMachine.name)"
                 break;
             }
            
             $Msg = ""
             if ($NetworkName) 
             {
                 $Network = Get-View -Id $HostSystem.Network -Property 'Name','Config' |
                     Where-Object { $_.Name -eq $NetworkName }
                 if ($Network) 
                 {
                     $Msg += "Connecting $($netAdapter.Name) to $($NetworkName) "
                     switch($Network.GetType().Name) 
                     {
                         "Network" 
                         {
                             $VirtualDeviceConfigSpec.device.backing = New-Object VMware.Vim.VirtualEthernetCardNetworkBackingInfo
                             $VirtualDeviceConfigSpec.device.backing.deviceName = $NetworkName
                         }
                         "DistributedVirtualPortgroup" 
                         {
                             $VirtualDeviceConfigSpec.device.Backing = New-Object VMware.Vim.VirtualEthernetCardDistributedVirtualPortBackingInfo
                             $VirtualDeviceConfigSpec.device.backing.port = New-Object VMware.Vim.DistributedVirtualSwitchPortConnection
                             $VirtualDeviceConfigSpec.device.backing.port.switchUuid = (Get-View $Network.Config.DistributedVirtualSwitch).Uuid
                             $VirtualDeviceConfigSpec.device.backing.port.portgroupKey = $Network.Config.Key
                         }
                     }
                 }
                 Else 
                 {
                     Write-Warning "$($NetworkName) was not found on $($HostSystem.Name)"
                 }
             }
            
             if ($PSCmdlet.MyInvocation.BoundParameters.Connected -ne $null) 
             {
                 $Msg += "Connected:$($Connected) "
                 $VirtualDeviceConfigSpec.Device.Connectable.Connected = $Connected
             }
             Else {
                 $VirtualDeviceConfigSpec.Device.Connectable.Connected = $netAdapter.ConnectionState.Connected
             }
            
             if ($PSCmdlet.MyInvocation.BoundParameters.StartConnected -ne $null) 
             {
                 $Msg += "StartConnected:$($StartConnected) "
                 $VirtualDeviceConfigSpec.Device.Connectable.StartConnected = $StartConnected
             }
             Else 
             {
                 $VirtualDeviceConfigSpec.Device.Connectable.StartConnected = $netAdapter.ConnectionState.StartConnected
             }
                
             $VirtualMachineConfigSpec = New-Object VMware.Vim.VirtualMachineConfigSpec
             $VirtualMachineConfigSpec.deviceChange += $VirtualDeviceConfigSpec
            
             IF ($PSCmdlet.ShouldProcess($VirtualMachine.Name,$Msg)) 
             {
                 $Task = Get-VIObjectByVIView -MORef $VirtualMachine.ReconfigVM_Task($VirtualMachineConfigSpec)
                
                 if (-Not $RunAsync) 
                 {
                     Wait-Task -Task $Task | Out-Null
                     $returnNetworkAdapter = $netAdapter
                     $returnNetworkAdapter.NetworkName = $NetworkName
                     $returnNetworkAdapter.ConnectionState.StartConnected = $VirtualDeviceConfigSpec.Device.Connectable.StartConnected
                     $returnNetworkAdapter.ConnectionState.Connected = $VirtualDeviceConfigSpec.Device.Connectable.Connected
                     $returnNetworkAdapter
                 }
                 else 
                 {
                     $task
                 }           
             }
         }
     }
 }
 
 New-Alias -Name Get-NetworkAdapter -Value Get-dvSwitchNetworkAdapter
 New-Alias -Name Set-NetworkAdapter -Value Set-dvSwitchNetworkAdapter
 
 Export-ModuleMember -Function Get-dvSwitchNetworkAdapter, Set-dvSwitchNetworkAdapter -Alias Get-NetworkAdapter,Set-NetworkAdapter
`

