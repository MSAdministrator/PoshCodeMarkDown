---
Author: poshman
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6492
Published Date: 2016-08-29t07
Archived Date: 2016-09-04t23
---

# lost operators - 

## Description

adds imitations of some operators such as ternary operator, -all, -any and etc.

## Comments



## Usage



## TODO



## script

`invoke-lostoperator`

## Code

`#
 #
 
 
 Set-Alias ~! Invoke-LostOperator
 
 function Invoke-LostOperator {
   <#
     .SYNOPSIS
         Compensates lack of some operators.
     .DESCRIPTION
         This function has alias "~!". It seems the best way to warn user about that an
         operator does not really exist.
     .EXAMPLE
         PS C:\> ~! @(1, 2, 3, 4, 5) -any '($x % 2) -eq 0'
         True
     .EXAMPLE
         PS C:\> ~! @(2, 4, 6, 8, 10) -all '($x % 2) -eq 0'
         False
     .EXAMPLE
         PS C:\> ~! @('this', 'is', 'array', 1, 2, 3) -first '$x.GetTtpe().Name -eq "Int32"'
         1
     .EXAMPLE
         PS C:\> ~! @('this', 'is', 'array', 1, 2, 3) -last '$x.GetType().Name -eq "String"'
         array
     .EXAMPLE
         PS C:\> ~! 7 -shl 3
         56
     .EXAMPLE
         PS C:\> ~! 7 -shr 1
         3
     .EXAMPLE
         PS C:\> ~! @('this', 'is', 'array', 1, 2, 3) -skip 3
         1
         2
         3
     .EXAMPLE
         PS C:\> ~! @('this', 'is', 'array', 1, 2, 3) -take 3
         this
         is
         array
     .EXAMPLE
         PS C:\> ~! ($x -gt 30) ? { $x + 5 } : { $x - 3 }
     .NOTES
         Operators -shl and -shr are not required for usage on PowerShell -ge 3.
         
         Alternative way to simulate a shift method on PowerShell -eq 2:
         PS C:\> $(cmd /c set /a 7 ^<^< 3);''
         56
         PS C:\> $(cmd /c set /a 7 ^>^> 1);''
         3
         
         To get all PowerShell -eq 2 operators:
         if ($PSVersionTable.PSVersion.Major -eq 2) {
           [PSObject].Assembly.GetType(
             'System.Management.Automation.OperatorTokenReader'
           ).GetFields([Reflection.BindingFlags]40) |
           Where-Object { $_.Name -match 'operators\Z' } |
           Sort-Object Name | ForEach-Object {
             Write-Host "$($_.Name.Replace(
               'Operators', ''
             )): " -ForegroundColor Green -NoNewLine
             Write-Host (
               $_.GetValue($null) -join ', '
             ) -ForegroundColor Yellow
           }
         }
   #>
   begin {
     if ($PSVersionTable.PSVersion.Major -eq 2) {
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
       
       function private:Set-Shift {
         param(
           [Parameter(Position=0)]
           [ValidateNotNullOrEmpty()]
           [ValidateSet('Left', 'Right')]
           [String]$Direction = 'Left',
           
           [Parameter(Position=1)]
           [ValidateNotNull()]
           [Object]$Type = [Int32]
         )
         
         @(
           'Ldarg_0'
           'Ldarg_1'
           'Ldc_I4_S, 31'
           'And'
           $(if ($Direction -eq 'Right') { 'Shr' } else { 'Shl' })
           'Ret'
         ) | ForEach-Object {
           $def = New-Object DynamicMethod(
             $Direction, $Type, @($Type, $Type)
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
     
     function private:Invoke-Eval {
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
     
     function private:Invoke-Linq {
       param(
         [Parameter(Mandatory=$true, Position=0)]
         [ValidateNotNullOrEmpty()]
         [String]$Method,
         
         [Parameter(Mandatory=$true, Position=1)]
         [ValidateNotNull()]
         [Object[]]$Collection,
         
         [Parameter(Mandatory=$true, Position=2)]
         [ValidateNotNullOrEmpty()]
         [String]$Condition,
         
         [Parameter(Position=3)]
         [Type[]]$Type = [Object]
       )
       
       $m = ([Linq.Enumerable].GetMember(
         $Method, [Reflection.MemberTypes]8,
         [Reflection.BindingFlags]24
       ) | Where-Object {
         $_.IsGenericMethod -and $_.GetParameters(
         ).Length -eq 2
       }).MakeGenericMethod($Type)
       
       switch -Regex ($Method) {
         '\A(Skip|Take)\Z' {
           $i = 0
           if (![Int32]::TryParse($Condition, [ref]$i)) {
             return $null
           }
           
           $m.Invoke($null, @($Collection, $i))
         }
         '\A(All|Any|First|Last)\Z' {
           $e = Invoke-Expression "[Func[$Type, Boolean]]{param(
             $(([Regex]'\$\w+').Match($Condition).Value)
           ) $Condition}"
           
           $m.Invoke($null, @($Collection, $e))
         }
       }
     }
   }
   process {
     if ($args) {
       $l = [Array]::IndexOf($args, $args[1]) + 1
       switch ($args.Length) {
         3 {
           switch -Regex ($args[1]) {
             '\A-(any|all|first|last|skip|take)\Z' {
               return Invoke-Linq "$([Char]::ToUpper($args[1][1])
               )$(-join $args[1][2..$args[1].Length])" $args[0] $args[$l]
             }
               if ($collect) {
                 return (Set-Shift $(switch (
                   $args[1][-1]) {'l'{'Left'}'r'{'Right'}})
                 ).Invoke($args[0], $args[$l])
               }
             }
           }
         5 {
           if (Invoke-Eval $args[0]) {
             return Invoke-Eval $args[$l]
           }
           return Invoke-Eval ($args[[Array]::IndexOf($args, ':', $l) + 1])
       }
       
       return Invoke-Eval $args[0]
     }
   }
   end {
     if ($collect) { $collect | ForEach-Object { [void]$ta::Remove($_) } }
   }
 }
`

