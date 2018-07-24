---
Author: hacker
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6469
Published Date: 2016-08-10t19
Archived Date: 2016-08-13t22
---

# new-password - 

## Description

re\sets password for specified user. original https

## Comments



## Usage



## TODO



## function

`new-password`

## Code

`#
 #
 function New-Password {
   <#
     .SYNOPSIS
         Changes password for specified user.
     .EXAMPLE
         PS C:\> New-Password oldpass newpass Administrator
   #>
   param(
     [Parameter(Mandatory=$true, Position=0)]
     [ValidateNotNullOrEmpty()]
     [String]$OldPassword,
     
     [Parameter(Mandatory=$true, Position=1)]
     [ValidateNotNullOrEmpty()]
     [String]$NewPassword,
     
     [Parameter(Mandatory=$true, Position=2)]
     [ValidateNotNullOrEmpty()]
     [String]$UserName,
     
     [Parameter(Position=4)]
     [AllowEmptyString()]
     [String]$Domain = $null
   )
   
   begin {
     @(
       [Runtime.InteropServices.CallingConvention],
       [Runtime.InteropServices.HandleRef],
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
         
         [Regex].Assembly.GetType(
           'Microsoft.Win32.UnsafeNativeMethods'
         ).GetMethods() | Where-Object {
           $_.Name -cmatch '\AGet(ProcA|ModuleH)'
         } | ForEach-Object {
           Set-Variable $_.Name $_
         }
       }
       process {
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
         }
         
         if (($ptr = $GetProcAddress.Invoke($null, @(
           [HandleRef](New-Object HandleRef(
              (New-Object IntPtr),
              $GetModuleHandle.Invoke($null, @($Module)))
           ), $Function
         ))) -eq [IntPtr]::Zero) {
           Write-Warning "$(Get-LastError)"
           break
         }
         
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
           4 { $il.Emit([OpCodes]::Ldc_I4, $ptr.ToInt32()) }
           8 { $il.Emit([OpCodes]::Ldc_I8, $ptr.ToInt64()) }
         }
         $il.EmitCalli(
           [OpCodes]::Calli, [CallingConvention]::StdCall, $returntype, $paramtypes
         )
         $il.Emit([OpCodes]::Ret)
       }
       end { $holder.CreateDelegate($proto), $mod }
     }
     
     $NetUserChangePassword, $mod = Set-Delegate netapi32 NetUserChangePassword `
                            '[Func[[Byte[]], [Byte[]], [Byte[]], [Byte[]], UInt32]]'
   }
   process {
     $OldPassword, $NewPassword, $UserName, $Domain | ForEach-Object { $i = 1 }{
       Set-Variable "p$i" $([Text.Encoding]::Unicode.GetBytes($_))
       $i++
     }
     if (($err = $NetUserChangePassword.Invoke($p4, $p3, $p1, $p2)) -ne 0) {
       Write-Warning "could not change password, errror: $('0x{0:X}' -f $err)"
     }
   }
   end {
     if ($mod) {
       [void][Linq.Enumerable].Assembly.GetType(
         'Microsoft.Win32.UnsafeNativeMethods'
       ).GetMethod(
         'FreeLibrary', [BindingFlags]40
       ).Invoke($null, @($mod))
     }
     $collect | ForEach-Object { [void]$ta::Remove($_) }
   }
 }
`

