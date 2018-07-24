---
Author: matthew graeber
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3859
Published Date: 2013-01-03t03
Archived Date: 2013-01-09t07
---

# get-structfrommemory - 

## Description

marshals data from an unmanaged block of memory in an arbitrary process to a newly allocated managed object of the specified type.

## Comments



## Usage



## TODO



## function

`get-structfrommemory`

## Code

`#
 #
 function Get-StructFromMemory
 {
 <#
 .SYNOPSIS
 
 Marshals data from an unmanaged block of memory in an arbitrary process to a newly allocated managed object of the specified type.
 
 Author: Matthew Graeber (@mattifestation)
 License: BSD 3-Clause
  
 .DESCRIPTION
 
 Get-StructFromMemory is similar to the Marshal.PtrToStructure method but will parse and return a structure from any process.
 
 .PARAMETER Id
 
 Process ID of the process whose virtual memory space you want to access.
 
 .PARAMETER MemoryAddress
 
 The address containing the structure to be parsed.
 
 .PARAMETER StructType
 
 The type (System.Type) of the desired structure to be parsed.
 
 .EXAMPLE
 
 C:\PS> Get-Process | ForEach-Object { Get-StructFromMemory -Id $_.Id -MemoryAddress $_.MainModule.BaseAddress -StructType ([PE+_IMAGE_DOS_HEADER]) }
 
 Description
 -----------
 Parses the DOS headers of every loaded process. Note: In this example, this assumes that [PE+_IMAGE_DOS_HEADER] is defined. You can get the code to define [PE+_IMAGE_DOS_HEADER] here: http://www.exploit-monday.com/2012/07/structs-and-enums-using-reflection.html
 
 .NOTES
 
 Be sure to enclose the StructType parameter with parenthesis in order to force PowerShell to cast it as a Type object.
 
 Get-StructFromMemory does a good job with error handling however it will crash if the structure contains fields that attempt to marshal pointers. For example, if a field has a custom attribute of UnmanagedType.LPStr, when the structure is parsed, it will attempt to dererence a string pointer for virtual memory in another process and access violate.
 
 .LINK
 
 http://www.exploit-monday.com/
 #>
 
     [CmdletBinding()] Param (
         [Parameter(Position = 0, Mandatory = $True)]
         [Alias('ProcessId')]
         [Alias('PID')]
         [UInt16]
         $Id,
 
         [Parameter(Position = 1, Mandatory = $True)]
         [IntPtr]
         $MemoryAddress,
 
         [Parameter(Position = 2, Mandatory = $True)]
         [Alias('Type')]
         [Type]
         $StructType
     )
 
     Set-StrictMode -Version 2
 
 
     $GetProcessHandle = [Diagnostics.Process].GetMethod('GetProcessHandle', [Reflection.BindingFlags] 'NonPublic, Instance', $null, @([Int]), $null)
 
     try
     {
         $Process = Get-Process -Id $Id -ErrorVariable GetProcessError
         $Handle = $Process.Handle
     }
     catch [Exception]
     {
         throw $GetProcessError
     }
 
     if ($Handle -eq $null)
     {
         throw "Unable to obtain a handle for PID $Id. You will likely need to run this script elevated."
     }
 
     $mscorlib = [AppDomain]::CurrentDomain.GetAssemblies() | ? { $_.FullName.Split(',')[0].ToLower() -eq 'mscorlib' }
     $Win32Native = $mscorlib.GetTypes() | ? { $_.FullName -eq 'Microsoft.Win32.Win32Native' }
     $MEMORY_BASIC_INFORMATION = $Win32Native.GetNestedType('MEMORY_BASIC_INFORMATION', [Reflection.BindingFlags] 'NonPublic')
 
     if ($MEMORY_BASIC_INFORMATION -eq $null)
     {
         throw 'Unable to get a reference to the MEMORY_BASIC_INFORMATION structure.'
     }
 
     $ProtectField = $MEMORY_BASIC_INFORMATION.GetField('Protect', [Reflection.BindingFlags] 'NonPublic, Instance')
     $AllocationBaseField = $MEMORY_BASIC_INFORMATION.GetField('BaseAddress', [Reflection.BindingFlags] 'NonPublic, Instance')
     $RegionSizeField = $MEMORY_BASIC_INFORMATION.GetField('RegionSize', [Reflection.BindingFlags] 'NonPublic, Instance')
 
     {
         $DynAssembly = New-Object Reflection.AssemblyName('MemHacker')
         $AssemblyBuilder = [AppDomain]::CurrentDomain.DefineDynamicAssembly($DynAssembly, [Reflection.Emit.AssemblyBuilderAccess]::Run)
         $ModuleBuilder = $AssemblyBuilder.DefineDynamicModule('MemHacker', $False)
         $Attributes = 'AutoLayout, AnsiClass, Class, Public, SequentialLayout, Sealed, BeforeFieldInit'
         $TypeBuilder = $ModuleBuilder.DefineType('NativeUtils', $Attributes, [ValueType])
         $TypeBuilder.DefinePInvokeMethod('ReadProcessMemory', 'kernel32.dll', [Reflection.MethodAttributes] 'Public, Static', [Reflection.CallingConventions]::Standard, [Bool], @([IntPtr], [IntPtr], [IntPtr], [UInt32], [UInt32].MakeByRefType()), [Runtime.InteropServices.CallingConvention]::Winapi, 'Auto') | Out-Null
         $TypeBuilder.DefinePInvokeMethod('VirtualQueryEx', 'kernel32.dll', [Reflection.MethodAttributes] 'Public, Static', [Reflection.CallingConventions]::Standard, [UInt32], @([IntPtr], [IntPtr], $MEMORY_BASIC_INFORMATION.MakeByRefType(), [UInt32]), [Runtime.InteropServices.CallingConvention]::Winapi, 'Auto') | Out-Null
 
         $NativeUtils = $TypeBuilder.CreateType()
     }
 
     try
     {
         $SafeHandle = $GetProcessHandle.Invoke($Process, @($PROCESS_VM_READ))
         $Handle = $SafeHandle.DangerousGetHandle()
     }
     catch
     {
         throw $Error[0]
     }
 
     $MemoryBasicInformation = [Activator]::CreateInstance($MEMORY_BASIC_INFORMATION)
 
     $NativeUtils::VirtualQueryEx($Handle, $MemoryAddress, [Ref] $MemoryBasicInformation, [Runtime.InteropServices.Marshal]::SizeOf($MEMORY_BASIC_INFORMATION)) | Out-Null
 
     $PAGE_EXECUTE_READ = 0x20
     $PAGE_EXECUTE_READWRITE = 0x40
     $PAGE_READONLY = 2
     $PAGE_READWRITE = 4
 
     $Protection = $ProtectField.GetValue($MemoryBasicInformation)
     $AllocationBaseOriginal = $AllocationBaseField.GetValue($MemoryBasicInformation)
     $GetPointerValue = $AllocationBaseOriginal.GetType().GetMethod('GetPointerValue', [Reflection.BindingFlags] 'NonPublic, Instance')
     $AllocationBase = $GetPointerValue.Invoke($AllocationBaseOriginal, $null).ToInt64()
     $RegionSize = $RegionSizeField.GetValue($MemoryBasicInformation).ToUInt64()
 
     Write-Verbose "Protection: $Protection"
     Write-Verbose "AllocationBase: $AllocationBase"
     Write-Verbose "RegionSize: $RegionSize"
 
     if (($Protection -ne $PAGE_READONLY) -and ($Protection -ne $PAGE_READWRITE) -and ($Protection -ne $PAGE_EXECUTE_READ) -and ($Protection -ne $PAGE_EXECUTE_READWRITE))
     {
         $SafeHandle.Close()
         throw 'The address specified does not have read access.'
     }
 
     $StructSize = [Runtime.InteropServices.Marshal]::SizeOf($StructType)
     $EndOfAllocation = $AllocationBase + $RegionSize
     $EndOfStruct = $MemoryAddress.ToInt64() + $StructSize
 
     if ($EndOfStruct -gt $EndOfAllocation)
     {
         $SafeHandle.Close()
         throw 'You are attempting to read beyond what was allocated.'
     }
 
     try
     {
         $LocalStructPtr = [Runtime.InteropServices.Marshal]::AllocHGlobal($StructSize)
     }
     catch [OutOfMemoryException]
     {
         throw Error[0]
     }
 
     Write-Verbose "Memory allocated at 0x$($LocalStructPtr.ToString("X$([IntPtr]::Size * 2)"))"
 
     $ZeroBytes = New-Object Byte[]($StructSize)
     [Runtime.InteropServices.Marshal]::Copy($ZeroBytes, 0, $LocalStructPtr, $StructSize)
 
     $BytesRead = [UInt32] 0
 
     if ($NativeUtils::ReadProcessMemory($Handle, $MemoryAddress, $LocalStructPtr, $StructSize, [Ref] $BytesRead))
     {
         $SafeHandle.Close()
         [Runtime.InteropServices.Marshal]::FreeHGlobal($LocalStructPtr)
         throw ([ComponentModel.Win32Exception][Runtime.InteropServices.Marshal]::GetLastWin32Error())
     }
 
     Write-Verbose "Struct Size: $StructSize"
     Write-Verbose "Bytes read: $BytesRead"
 
     $ParsedStruct = [Runtime.InteropServices.Marshal]::PtrToStructure($LocalStructPtr, $StructType)
 
     [Runtime.InteropServices.Marshal]::FreeHGlobal($LocalStructPtr)
     $SafeHandle.Close()
 
     Write-Output $ParsedStruct
 }
`

