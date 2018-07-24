---
Author: roger
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6475
Published Date: 2016-08-16t13
Archived Date: 2016-08-18t21
---

# get-ntversionnumbers - 

## Description

nice solution to detect current windows version (instead buggy [environment]

## Comments



## Usage



## TODO



## function

`get-ntversionnumbers`

## Code

`#
 #
 function Get-NtVersionNumbers {
   <#
     .SYNOPSIS
         Gets the version numbers of the run time library.
   #>
   
   begin {
     @(
       [Runtime.InteropServices.CallingConvention],
       [Runtime.InteropServices.HandleRef],
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
     
     function private:Set-Delegate {
       param(
         [Parameter(Mandatory=$true, Position=0)]
         [ValidateNotNullOrEmpty()]
         [String]$Module,
         
         [Parameter(Mandatory=$true, Position=1)]
         [ValidateNotNullOrEmpty()]
         [String]$Function,
         
         [Parameter(Mandatory=$true, Position=2)]
         [ValidateNotNullOrEmpty()]
         [String]$Delegate
       )
       
       begin {
         [Regex].Assembly.GetType(
           'Microsoft.Win32.UnsafeNativeMethods'
         ).GetMethods() | Where-Object {
           $_.Name -cmatch '\AGet(ProcA|ModuleH)'
         } | ForEach-Object {
           Set-Variable $_.Name $_
         }
         
         $ptr = $GetProcAddress.Invoke($null, @(
           [HandleRef](New-Object HandleRef(
             (New-Object IntPtr), $GetModuleHandle.Invoke($null, @($Module))
           )), $Function
         ))
       }
       process { $proto = Invoke-Expression $Delegate }
       end {
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
           [OpCodes]::Calli, [CallingConvention]::StdCall, $returntype, $paramtypes
         )
         $il.Emit([OpCodes]::Ret)
         
         $holder.CreateDelegate($proto)
       }
     }
     
     $RtlGetNtVersionNumbers = Set-Delegate ntdll RtlGetNtVersionNumbers `
                                           '[Action[[Byte[]], [Byte[]], [Byte[]]]]'
   }
   process {
     $maj, $min = [Byte[]]@(0, 0, 0, 0), [Byte[]]@(0, 0, 0, 0)
     $RtlGetNtVersionNumbers.Invoke($maj, $min, $null)
     ($maj, $min | ForEach-Object { [BitConverter]::ToUInt32($_, 0) }) -join '.'
   }
   end {
     $collect | ForEach-Object { [void]$ta::Remove($_) }
   }
 }
`

