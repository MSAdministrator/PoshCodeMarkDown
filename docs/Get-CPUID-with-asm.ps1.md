---
Author: jakub jares
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6405
Published Date: 2016-06-24t19
Archived Date: 2016-06-29t04
---

# get-cpuid (with asm) - 

## Description

original post was found at https

## Comments



## Usage



## TODO



## function

`get-cpuid`

## Code

`#
 #
 function Get-CPUID {
   <#
     .SYNOPSIS
         Queries the CPU for information about its type.
   #>
   begin {
     @(
       [Runtime.InteropServices.CallingConvention],
       [Runtime.InteropServices.GCHandle],
       [Runtime.InteropServices.Marshal],
       [Reflection.Emit.OpCodes]
     ) | ForEach-Object {
       $keys = ($ta = [PSObject].Assembly.GetType(
         'System.Management.Automation.TypeAccelerators'
       ))::Get.Keys
       $collect = @()
     }{
       if ($keys -notcontains $_.Name) {
         $ta::Add($_.Name, $_)
         $collect += $_.Name
       }
     }
     
     Add-Type -AssemblyName System.ServiceModel
     
     ([AppDomain]::CurrentDomain.GetAssemblies() |
     Where-Object {
       $_.ManifestModule.ScopeName.Equals(
         'System.ServiceModel.dll'
       )
     }).GetType(
       'System.ServiceModel.Channels.UnsafeNativeMethods'
     ).GetMethods([Reflection.BindingFlags]40) |
     Where-Object {
       $_.Name -cmatch '\AVirtual(Alloc|Free)'
     } | ForEach-Object { Set-Variable $_.Name $_ }
     
     [Byte[]]$bytes = switch ([IntPtr]::Size) {
       4 {
       }
       8 {
       }
     }
   }
   process {
     try {
       $ptr = $VirtualAlloc.Invoke($null, @(
         [IntPtr]::Zero, [UIntPtr](New-Object UIntPtr($bytes.Length)),
         [UInt32](0x1000 -bor 0x2000), [UInt32]0x40
       ))
       
       [Marshal]::Copy($bytes, 0, $ptr, $bytes.Length)
       
       $proto  = [Action[Int32, [Byte[]]]]
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
         4 { $il.Emit([OpCodes]::Ldc_I4, $ptr.ToInt32()) }
         8 { $il.Emit([OpCodes]::Ldc_I8, $ptr.ToInt64()) }
       }
       $il.EmitCalli(
         [OpCodes]::Calli, [CallingConvention]::Cdecl, $returntype, $paramtypes
       )
       $il.Emit([OpCodes]::Ret)
       
       $cpuid = $holder.CreateDelegate($proto)
       
       [Byte[]]$buf = New-Object Byte[] 16
       $gch = [GCHandle]::Alloc($buf, 'Pinned')
       $cpuid.Invoke(0, $buf)
       $gch.Free()
       
       "$(-join [Char[]]$buf[4..7])$(
             -join [Char[]]$buf[12..15]
       )$(-join [Char[]]$buf[8..11])"
     }
     catch { $_.Exception }
     finally {
       if ($ptr) {
         [void]$VirtualFree.Invoke($null, @($ptr, [UIntPtr]::Zero, [UInt32]0x8000))
       }
     }
   }
   end {
     $collect | ForEach-Object { [void]$ta::Remove($_) }
   }
 }
`

