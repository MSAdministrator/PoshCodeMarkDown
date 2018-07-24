---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 969
Published Date: 2011-03-20t07
Archived Date: 2011-05-04t04
---

# import-methods - 

## Description

add functions to the scope for each static method of a type. originally from oisin grehan

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 param(
    #[Parameter(Position=1,ValueFromPipeline=$true)]
    [type]$type
 ,
    #[Parameter()]
    #[Alias("Properties")]
    [switch]$IncludeProperties
 )
 BEGIN {
    function MakeFunction() {
     PROCESS {
       $func = "function:global:$($_.name)"
       if (test-path $func) { remove-item $func }
       $flags = 'Public,Static,InvokeMethod,DeclaredOnly'
       new-item $func -value "[$($type.fullname)].InvokeMember('$($_.name)',$flags, `$null, `$null, `$args[0])"
     } 
    } 
    function MakeVariable($type) {
     PROCESS {
       $var = "variable:global:$($_.name)"
       if (test-path $var) { Remove-Variable $var -ErrorAction SilentlyContinue }
       new-variable -Name $($_.name) -Value $(Invoke-Expression "[$($type.fullname)]::$($_.name)") `
                    -Description  "Imported from $($type.FullName)" -Force -Scope Global `
                    -Option AllScope, Constant, ReadOnly
     }
    }
 }
 PROCESS {
    if($_) { $type = $_ }
    $type | gm -static -membertype "method" | MakeFunction
 
    if($IncludeProperties) {
       $type | gm -static -membertype "property" | MakeVariable $type
    } 
 }
`

