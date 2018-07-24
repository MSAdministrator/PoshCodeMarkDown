---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6836
Published Date: 2017-04-08t18
Archived Date: 2017-04-09t13
---

# handles - 

## Description

original github link https

## Comments



## Usage



## TODO



## function

`get-handles`

## Code

`#
 #
 function Get-Handles {
   <#
     .SYNOPSIS
         Enumarates handles of the specified process.
     .EXAMPLE
         PS C:\> Get-Handles 3828
     .NOTES
         Author: greg zakharov
 
         PROCESS_DUP_HANDLE = 0x40
         STATUS_INFO_LENGTH_MISMATCH = 0xC0000004
         STATUS_SUCCESS = 0x00000000
 
         SystemHandleInformation = 16
         ObjectNameInformation = 1
         ObjectTypeInformation = 2
   #>
   param(
     [Parameter(Mandatory=$true)]
     [ValidateScript({($script:proc = Get-Process -Id $_ -ea 0) -ne $null})]
     [Int32]$Id
   )
 
   begin {
     @(
       [Runtime.InteropServices.Marshal],
       [Runtime.InteropServices.GCHandle],
       [Runtime.InteropServices.CharSet],
       [Runtime.InteropServices.CallingConvention],
       [Reflection.BindingFlags]
     ) | ForEach-Object {
       $keys = ($ta= [PSObject].Assembly.GetType(
         'System.Management.Automation.TypeAccelerators'
       ))::Get.Keys
       $collect = @()
     }{
       if ($keys -notcontains $_.Name) { $ta::Add($_.Name, $_) }
       $collect += $_.Name
     }
 
     function private:New-DllImport {
       param(
         [Parameter(Mandatory=$true, Position=0)]
         [ValidateNotNullOrEmpty()]
         [String]$Module,
 
         [Parameter(Mandatory=$true, Position=1)]
         [ValidateNotNull()]
         [Hashtable]$Signature,
 
         [Parameter(Position=2)]
         [String]$Type = ${Module}
       )
 
       begin {
         $mod = if (!($m = $ExecutionContext.SessionState.PSVariable.Get(
           'PowerShellDllImport'
         ))) {
           $mb = ([AppDomain]::CurrentDomain.DefineDynamicAssembly(
             (New-Object Reflection.AssemblyName('PowerShellDllImport')), 'Run'
           )).DefineDynamicModule('PowerShellDllImport', $false)
 
           Set-Variable PowerShellDllImport -Value $mb -Option Constant `
                                              -Scope Global -Visibility Private
         }
         else { $m.Value }
 
         ($dllimport = [Runtime.InteropServices.DllImportAttribute]).GetFields() |
         Where-Object { $_.Name -cmatch '\A(C|En|S)' } | ForEach-Object {
           Set-Variable "_$($_.Name)" $_
         }
       }
       process {}
       end {
         try { $pin = $mod.GetType("${Type}Handles") }
         catch {}
         finally {
           if (!$pin) {
             $pin = $mod.DefineType("${Type}Handles", 'Public, BeforeFieldInit')
 
             foreach ($func in $Signature.Keys) {
               <#
                 arr[0] - return type
                 arr[1] - parameters
                 arr[2] - other (SetLastError, CharSet and etc.)
               #>
               $arr = $Signature[$func]
               $fun = $pin.DefineMethod(
                 $func, 'Public, Static, PinvokeImpl', [Type]$arr[0], [Type[]]$arr[1]
               )
 
               $arr[1] | ForEach-Object { $i = 1 }{
                 if ($_.IsByRef) { [void]$fun.DefineParameter($i, 'Out', $null) }
               }
               $ErrorValue = if ($arr[2]) { $true } else { $false }
               $EntryPoint = if ($arr[3]) { $EntryPoint } else { $func }
 
               $atr = New-Object Reflection.Emit.CustomAttributeBuilder(
                 $dllimport.GetConstructor([String]), $Module,
                 [Reflection.PropertyInfo[]]@(), [Object[]]@(),
                 [Reflection.FieldInfo[]]@(
                   $_SetlastError, $_CallingConvention, $_CharSet, $_EntryPoint
                 ), [Object[]]@(
                   $ErrorValue, [CallingConvention]::WinApi, [CharSet]::Auto,
                   $EntryPoint
                 )
               )
               $fun.SetCustomAttribute($atr)
             }
             $pin = $pin.CreateType()
           $pin
         }
       }
     }
 
     function private:Move-Next([IntPtr]$ptr) {
       $mov = switch ([IntPtr]::Size) {4 {$ptr.ToInt32()} 8 {$ptr.ToInt64()}}
       [IntPtr]($mov + $hes)
     }
 
     $kernel32 = New-DllImport kernel32 -Signature @{
       CloseHandle = [Boolean], @([IntPtr]), $true
       OpenProcess = [IntPtr], @([UInt32], [Boolean], [Int32]), $true
       GetCurrentProcess = [IntPtr], @(), $true
     }
 
     $ntdll = New-DllImport ntdll -Signature @{
       NtDuplicateObject = [Int32], @(
         [IntPtr], [IntPtr], [IntPtr], [IntPtr].MakeByRefType(),
         [UInt32], [UInt32], [UInt32]
       )
       NtQueryObject = [Int32], @(
         [IntPtr], [UInt32], [IntPtr], [UInt32], [UInt32].MakeByRefType()
       )
       NtQuerySystemInformation = [Int32], @(
         [UInt32], [IntPtr], [UInt32], [UInt32].MakeByRefType()
       )
     }
 
     $usz = [Marshal]::SizeOf((
       $UNICODE_STRING = [Activator]::CreateInstance(
         [Object].Assembly.GetType(
           'Microsoft.Win32.Win32Native+UNICODE_STRING'
         )
       )
     ))
   }
   process {
 
     try {
       if ((
         $hndl = $kernel32::OpenProcess(0x40, $false, $proc.Id
       )) -eq [IntPtr]::Zero) {
         throw New-Object InvalidOperationException
       }
       $shi = [Marshal]::AllocHGlobal($sz)
       while ($ntdll::NtQuerySystemInformation(
         16, $shi, $sz, [ref]$ret
       ) -eq 0xC0000004) {
         $shi = [Marshal]::ReAllocHGlobal(
           $shi, [IntPtr]($sz *= 2)
         )
       }
       $noh = [Marshal]::ReadInt32($shi)
       $mov = switch ([IntPtr]::Size) {4 {$shi.ToInt32()} 8 {$shi.ToInt64()}}
       $tmp = [IntPtr]($mov + [Marshal]::SizeOf([Type][IntPtr]))
       $gao = switch ([IntPtr]::Size) {4 {0x0C} 8 {0x10}}
       $handles = for ($i = 0; $i -lt $noh; $i++) {
         $hte = New-Object PSObject -Property @{
           UniqueProcessId  = [Marshal]::ReadInt32($tmp)
           ObjectTypeIndex  = [Marshal]::ReadByte($tmp, 0x04)
           HandleAttributes = [Marshal]::ReadByte($tmp, 0x05)
           HandleValue      = [Marshal]::ReadInt16($tmp, 0x06)
           Object           = [Marshal]::ReadIntPtr($tmp, 0x08)
           GrantedAccess    = [Marshal]::ReadInt32($tmp, $gao)
         }
 
         if ($hte.UniqueProcessId -ne $Id) {
           $tmp = Move-Next $tmp
           continue
         }
         if ($ntdll::NtDuplicateObject(
           $hndl, [IntPtr]$hte.HandleValue, $kernel32::GetCurrentProcess(),
           [ref]$duple, 0, 0, 0
         ) -ne 0) {
           $tmp = Move-Next $tmp
           continue
         }
 
         try {
           $obj = [Marshal]::AllocHGlobal($page)
           if ($ntdll::NtQueryObject($duple, 2, $obj, $page, [ref]$ret) -ne 0) {
             throw New-Object InvalidOperationException
           }
 
           [Byte[]]$bytes = 0..($usz - 1) | ForEach-Object {$ofb = 0}{
             [Marshal]::ReadByte($obj, $ofb)
             $ofb++
           }
 
           $gch = [GCHandle]::Alloc($bytes, 'Pinned')
           $uni = [Marshal]::PtrToStructure(
             $gch.AddrOfPinnedObject(), [Type]$UNICODE_STRING.GetType()
           )
           $gch.Free()
           $obj_type = $uni.GetType().GetField('Buffer', [BindingFlags]36).GetValue($uni)
 
           if ($hte.GrantedAccess -eq 0x12019F) {
             throw New-Object InvalidOperationException
           }
           $ret = 0
           $name = [Marshal]::AllocHGlobal($page)
           if ($ntdll::NtQueryObject($duple, 1, $name, $page, [ref]$ret) -ne 0) {
             $name = [Marshal]::ReAllocHGlobal($name, [IntPtr]$ret)
             if ($ntdll::NtQueryObject($duple, 1, $name, $page, [ref]$ret) -ne 0) {
               throw New-Object InvalidOperationException
             }
           }
 
           $uni = [Marshal]::PtrToStructure($name, [Type]$UNICODE_STRING.GetType())
           $obj_name = $uni.GetType().GetField('Buffer', [BindingFlags]36).GetValue($uni)
           if (![String]::IsNullOrEmpty($obj_name)) {
             New-Object PSObject -Property @{
               Handle = '0x{0:X}' -f $hte.HandleValue
               Type   = $obj_type
               Name   = $obj_name
             }
           }
         }
         catch {}
         finally {
           if ($name -ne [IntPtr]::Zero) { [Marshal]::FreeHGlobal($name) }
           if ($obj -ne [IntPtr]::Zero) { [Marshal]::FreeHGlobal($obj) }
         }
 
         [void]$kernel32::CloseHandle($duple)
         $tmp = Move-Next $tmp
       }
     }
     catch { $_.Exception }
     finally {
       if ($shi) { [Marshal]::FreeHGlobal($shi) }
       if ($hndl) { [void]$kernel32::CloseHandle($hndl) }
     }
   }
   end {
     if ($handles) { $handles | Format-List }
     $collect | ForEach-Object { [void]$ta::Remove($_) }
   }
 }
`

