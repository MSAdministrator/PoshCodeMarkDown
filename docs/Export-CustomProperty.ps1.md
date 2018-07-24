---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2269
Published Date: 
Archived Date: 2010-10-04t16
---

# export-customproperty - 

## Description

an example of making “custom objects” with strongly-typed properties, custom enumeration/validation and error messages.

## Comments



## Usage



## TODO



## function

`export-customproperty`

## Code

`#
 #
 
 function Export-CustomProperty {
 param(
    [Parameter(ValueFromPipeline=$true)]
    [PSCustomObject]$Object
 )
 process {
    foreach($property in Get-Member Get_*,Set_* -Input $Object -Type Methods | Group-Object { $_.Name.Split("_",2)[1] }) {
       $GetAccessor = $SetAccessor = {}
       foreach($accessor in $property.Group) {
          switch($accessor.Name.Split("_")[0]) {
             "Get" {
                $GetAccessor = [ScriptBlock]::Create('$this.' + $accessor.Name + '()')
             }
             "Set" {
                $SetAccessor = [ScriptBlock]::Create('try { $this.' + $accessor.Name + '( $args ) } catch { throw $_.Exception.Message.Split(":",2)[1].trim('' "'') }')
             }
          }
       }
          
       Add-Member -Input $Object -Type ScriptProperty -Name $Property.Name -Value $GetAccessor $SetAccessor
    }
    Write-Output $Object
 }
 }
 
 
 
 $customThing = New-Module -AsCustomObject {
 
    [string]$FooBar = 'foo'
    [string[]]$FooBarValues = 'foo','bar'
    
    function Get_FooBar { $FooBar }
    
    function Set_FooBar { 
       param([string]$value)
       if($FooBarValues -notcontains $Value) {
          throw "You can't set FooBar to '$value', valid values are $($FooBarValues -join ', ').";
       }
       $script:FooBar = $value
    }
    
    [bool]$Enabled = $False
    
   
    [String]$UserName = $Env:UserName
 
    function Test-ADUser {
       [CmdletBinding()]
       param([string]$UserName)
       $ads = New-Object System.DirectoryServices.DirectorySearcher([ADSI]'')
       $ads.filter = "(&(objectClass=Person)(samAccountName=$UserName))"
       return [bool]$ads.FindOne()
    }
 
    function Get_UserName { $UserName }
 
    function Set_UserName { 
       param([string]$value)
       if(!(Test-ADUser $Value)) {
          throw "You can't set UserName to '$value', that user doesn't exist.";
       }
       $script:UserName = $value
    }
 
    Export-ModuleMember -Function Get_*,Set_* -Variable Enabled
 
 } | Export-CustomProperty
 
 
 ##############################################################################
`

