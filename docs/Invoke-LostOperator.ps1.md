---
Author: roger
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6467
Published Date: 2016-08-07t16
Archived Date: 2016-08-09t11
---

# invoke-lostoperator - 

## Description

cool script which allows you to do some things more easily. for example, it became possible use bitwise shift in powershell v2 or c-style ternary operation.

## Comments



## Usage



## TODO



## function

`invoke-lostoperator`

## Code

`#
 #
 Set-Alias ~! Invoke-LostOperator
 
 function Invoke-LostOperator {
   <#
     .SYNOPSIS
         Compensates lack of some operators.
     .EXAMPLE
         PS C:\> Invoke-LostOperator { 1 + 1 }
         2
     .EXAMPLE
         PS C:\> ~! { 1 + 1 }
         2
         
         Same that's above but shortly.
     .EXAMPLE
         PS C:\> ~! 7 -shl 3
         56
         
         Note that shift operations works only in PowerShell v2, on
         higher version there are native operators -shl and -shr.
     .EXAMPLE
         PS C:\> ~! 7 -shr 1
         3
     .EXAMPLE
         PS C:\> ~! (100 -lt (Get-ChildItem).Count) ? 'yes' : 'no'
         no
         
         You can also use script blocks in ternary operation.
     .NOTES
         This function is experimental that's why there are some
         possible issues.
   #>
   begin {
     if (($maj = $PSVersionTable.PSVersion.Major) -eq 2) {
       @(
         [Reflection.Emit.DynamicMethod],
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
       
       function private:Set-ShiftMethod {
         param(
           [Parameter(Position=0)]
           [ValidateNotNullOrEmpty()]
           [ValidateSet('Left', 'Right')]
           [String]$X = 'Left',
           
           [Parameter(Position=1)]
           [ValidateNotNull()]
           [Object]$Type = [Int32]
         )
         
         @(
           'Ldarg_0'
           'Ldarg_1'
           'Ldc_I4_S, 31'
           'And'
           $(if ($X -eq 'Right') { 'Shr' } else { 'Shl' })
           'Ret'
         ) | ForEach-Object {
           $def = New-Object DynamicMethod(
             $X, $Type, @($Type, $Type)
           )
           $il = $def.GetILGenerator()
         }{
           if ($_ -notmatch ',') { $il.Emit([OpCodes]::$_) }
           else {
             $il.Emit(
               [OpCodes]::(($$ = $_.Split(','))[0]), ($$[1].Trim() -as $Type)
           )}
         }
         
         $def.CreateDelegate((
           Invoke-Expression "[Func[$($Type.Name), $($Type.Name), $($Type.Name)]]"
         ))
       }
     }
     
     function private:eval {
       param(
         [Parameter(Mandatory=$true)]
         [Object]$Object
       )
       
       if ($Object -ne $null) {
         if ($Object -is 'ScriptBlock') {
           return &$Object
         }
         return $Object
       }
       
       return $null
     }
   }
   process {
     if ($args) {
       if ($ta) {
         if ($l = [Array]::IndexOf($args, '-shl') + 1) {
           return (Set-ShiftMethod Left).Invoke($args[0], $args[$l])
         }
         if ($l = [Array]::IndexOf($args, '-shr') + 1) {
           return (Set-ShiftMethod Right).Invoke($args[0], $args[$l])
         }
       }
       if ($l = [Array]::IndexOf($args, '?') + 1) {
         if (eval($args[0])) {
           return eval($args[$l])
         }
         return eval($args[[Array]::IndexOf($args, ':', $l) + 1])
       }
       
       return eval($args[0])
     }
   }
   end {
     if ($ta) {
       $collect | ForEach-Object { [void]$ta::Remove($_) }
     }
   }
 }
`

