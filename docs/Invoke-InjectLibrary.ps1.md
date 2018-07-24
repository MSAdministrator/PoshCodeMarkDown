---
Author: dan jones
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6370
Published Date: 2016-06-06t18
Archived Date: 2016-06-08t01
---

# invoke-injectlibrary - 

## Description

original post of gregzakh can be found at https

## Comments



## Usage



## TODO



## function

`invoke-injectlibrary`

## Code

`#
 #
 function Invoke-InjectLibrary {
   <#
     .SYNOPSIS
         Injects specified dll into the target process.
     .NOTES
         This script has been written and tested on Win7 x86.
   #>
   param(
     [Parameter(Mandatory=$true, Position=0)]
     [ValidateScript({(Get-Process -Id $_ -ea 0) -ne $null})]
     [Int32]$Id,
     
     [Parameter(Mandatory=$true, Position=1)]
     [ValidateScript({Test-Path $_})]
     [String]$DllPath
   )
   
   begin {
     @(
       [Runtime.InteropServices.CallingConvention],
       [Reflection.Emit.OpCodes]
     ) | ForEach-Object {
       $keys = ($ta = [PSObject].Assembly.GetType(
         'System.Management.Automation.TypeAccelerators'
       ))::Get.Keys
     }{
       if ($keys -notcontains $_.Name) {
         $ta::Add($_.name, $_)
       }
     }
     
     function private:Get-ProcAddress {
       param(
         [Parameter(Mandatory=$true)]
         [String]$Function
       )
       
       [Object].Assembly.GetType(
         'Microsoft.Win32.Win32Native'
       ).GetMethods(
         [Reflection.BindingFlags]40
       ).Where{
         $_.Name -cmatch '\AGet(ProcA|ModuleH)'
       }.ForEach{
         Set-Variable $_.Name $_
       }
       
       $GetProcAddress.Invoke($null, @(
         $GetModuleHandle.Invoke(
           $null, @('kernel32.dll')
         ), $Function
       ))
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
       (0..($paramtypes.Length - 1)).ForEach{
         $il.Emit([OpCodes]::Ldarg, $_)
       }
       
       switch ([IntPtr]::Size) {
         4 { $il.Emit([OpCodes]::Ldc_I4, $ProcAddress.ToInt32()) }
         8 { $il.Emit([OpCodes]::Ldc_I8, $ProcAddress.ToInt64()) }
       }
       $il.EmitCalli(
         [OpCodes]::Calli, [CallingConvention]::StdCall,
         $returntype, $paramtypes
       )
       $il.Emit([OpCodes]::Ret)
       
       $holder.CreateDelegate($proto)
     }
     
     ('CloseHandle', 'CreateRemoteThread', 'LoadLibraryW', 'VirtualFreeEx',
     'OpenProcess', 'VirtualAllocEx', 'WaitForSingleObject', 'WriteProcessMemory'
     ).ForEach{
       Set-Variable $_ (Get-ProcAddress $_)
     }
     
     $CloseHandle = Set-Delegate $CloseHandle '[Func[IntPtr, Boolean]]'
     $OpenProcess = Set-Delegate $OpenProcess '[Func[UInt32, Boolean, Int32, IntPtr]]'
     $VirtualAllocEx = Set-Delegate $VirtualAllocEx `
                               '[Func[IntPtr, IntPtr, Int32, UInt32, Uint32, IntPtr]]'
     $VirtualFreeEx = Set-Delegate $VirtualFreeEx `
                                      '[Func[IntPtr, IntPtr, Int32, UInt32, Boolean]]'
     $CreateRemoteThread = Set-Delegate $CreateRemoteThread `
            '[Func[IntPtr, IntPtr, UInt32, IntPtr, IntPtr, UInt32, [Byte[]], IntPtr]]'
     $WaitForSingleObject = Set-Delegate $WaitForSingleObject `
                                                      '[Func[IntPtr, UInt32, UInt32]]'
     $WriteProcessMemory = Set-Delegate $WriteProcessMemory `
                            '[Func[IntPtr, IntPtr, [Byte[]], Int32, IntPtr, Boolean]]'
     $DllPath = Resolve-Path $DllPath
     $INFINITE = [BitConverter]::ToUInt32([BitConverter]::GetBytes(0xFFFFFFFF), 0)
   }
   process {
     if (($proc = $OpenProcess.Invoke(0x42A, $false, $Id)) -ne [IntPtr]::Zero) {
       if (($lrem = $VirtualAllocEx.Invoke($proc, [IntPtr]::Zero, $sz, 0x1000, 0x4)
       ) -ne [IntPtr]::Zero) {
         $bytes = [Text.Encoding]::Unicode.GetBytes($DllPath)
         if ($WriteProcessMemory.Invoke($proc, $lrem, $bytes, $sz, [IntPtr]::Zero)) {
           $bytes = [Byte[]]@(0, 0, 0, 0)
           if (($thrd = $CreateRemoteThread.Invoke(
             $proc, [IntPtr]::Zero, 0, $LoadLibraryW, $lrem, 0, $bytes
           )) -ne [IntPtr]::Zero) {
             [void]$WaitForSingleObject.Invoke($thrd, $INFINITE)
             [void]$CloseHandle.Invoke($thrd)
           }
         }
         [void]$VirtualFreeEx.Invoke($proc, $lrem, 0, 0x8000)
       }
       [void]$CloseHandle.Invoke($proc)
     }
   }
   end {
     ('OpCodes', 'CallingConvention').ForEach{
       [void]$ta::Remove($_)
     }
   }
 }
`

