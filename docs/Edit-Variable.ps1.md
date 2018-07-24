---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2471
Published Date: 
Archived Date: 2011-01-25t21
---

# edit-variable - 

## Description

a way to easily edit textual variables and environment variables

## Comments



## Usage



## TODO



## function

`edit-variable`

## Code

`#
 #
 function Edit-Variable {
     #.Parameter name
     #.Parameter Environment
     #.Example
     #.Example
     #.Example
     #.Example
 
     param(
         [Parameter(Position=0,ValueFromPipelineByPropertyName=$true,ValueFromPipeline=$true)]
         [string]$name
     ,
         [switch]$Environment
     )
     process {
         $path = Resolve-Path $name -ErrorAction SilentlyContinue
         if($Environment) {
             if(!$path -or $Path.Provider.Name -ne "Environment") {
                 $path = Resolve-Path "Env:$name"
             }
         } else {
             if($Path -and $Path.Provider.Name -eq "Environment") {
                 $Environment = $true
             } elseif(!$path -or $Path.Provider.Name -ne "Variable") {
                 $path = Resolve-Path "Variable:$name" -ErrorAction SilentlyContinue
             }
         }
         
         $temp = [IO.Path]::GetTempFileName()
         if($path) {
             if(!$Environment) {
                 $value = (Get-Variable $path.ProviderPath).Value
                 $string = $value -is [String]
                 if(!$string) {
                     Write-Warning "Variable $name is not a string variable, editing as CliXml"
                     Export-Clixml $temp -InputObject $Value 
                 } else {
                     Set-Content $temp $Value
                 }
             } else {
                 Get-Content $path | Set-Content $temp
             }
         } else {
             $Environment = $false
             New-Variable $Name
             $path = Resolve-Path Variable:$name -ErrorAction SilentlyContinue
         }
         if(!$path) {
             Write-Error "Cannot find variable '$name' because it does not exist."
         } else {
             $pre = Get-ChildItem $temp
             (Start-Process notepad $temp -passthru).WaitForExit()
             $post = Get-ChildItem $temp
             if($post.LastWriteTime -gt $pre.LastWriteTime) {
                 if(!$Environment) {
                     if(!$string) {
                         Import-CliXml $temp | Set-Variable $path.ProviderPath
                     } else {
                         Get-Content $temp | Set-Variable $path.ProviderPath
                     }
                 } else {
                     Get-Content $temp | Set-Content $path
                 }
             }
         }
         Remove-Item $temp -ErrorAction SilentlyContinue
     }
 }
 
 Set-Alias vared Edit-Variable
`

