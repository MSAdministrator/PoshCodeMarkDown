---
Author: marc carter
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2598
Published Date: 2011-04-04t19
Archived Date: 2011-04-10t07
---

# advanced1_2011.ps1 - 

## Description

posh entry

## Comments



## Usage



## TODO



## script

`get-moduleinfo`

## Code

`#
 #
 #--------------------------------------------------------
 #--------------------------------------------------------
 
                                                                      
                                                                      
                                                                      
                                              
 function get-ModuleInfo { 
 .SYNOPSIS   
     Retrieves module information for installed components.
 .DESCRIPTION 
     Retrieves module information for installed components.
 .PARAMETER server
     Name of the system you wish to query.
 .PARAMETER UseDefaultCredentials 
     Use the currently authenticated user's credentials   
 .NOTES   
     Name: get-ModuleInfo
     Author: Marc Carter
     DateCreated: 4APR2011
 .EXAMPLE   
     get-ModuleInfo "computer1"
      
 Description 
 ------------ 
 Returns Module properties for a given process. 
 #>  
 [cmdletbinding( 
     DefaultParameterSetName = 'server', 
     ConfirmImpact = 'low' 
 )] 
 Param( 
     [Parameter(
         Position = 0, 
         Mandatory = $True, 
         ParameterSetName = 'server', 
         ValueFromPipeline = $True,
         ValueFromPipelineByPropertyName = $True)] 
         [String][ValidatePattern(".{2,}")]$server
 ) 
 
     Begin{
         $psBoundParameters.GetEnumerator() | % {
             Write-Verbose "Parameter: $_"
         }
         $server = $server.toUpper()
     }
 
     Process{
         Write-Host `"Computer`"`,`"ModuleName`"`,`"Size`"`,`"FileName`"`,`"FileVersion`"
         if(($server -ne "localhost") -and ($server -ne $env:computername)){
             Invoke-Command -ComputerName $server -ScriptBlock { notepad }
             $proc = Get-Process -ComputerName $server
             $proc | Where-Object { $_.name -eq 'notepad' } | % {
                 $_.Modules | Where-Object {$_.Description -eq "Windows Spooler Driver" } | % {
                     [string]$ModuleName = $_.ModuleName
                     [string]$Size = $_.Size
                     [string]$FileName = $_.FileName
                     [string]$FileVersion = $_.FileVersion
                     Write-Host `"$server`"`,`"$ModuleName`"`,`"$Size`"`,`"$FileName`"`,`"$FileVersion`"
                 }
                 Stop-Process -Id $_.Id
             }
         }
         else{
             Invoke-Command -ScriptBlock { notepad }
             $proc = Get-Process -ComputerName $server
             $proc | Where-Object { $_.name -eq 'TextPad' } | % {
                 $_.Modules | Where-Object {$_.Description -eq "TextPad" } | % {
                     [string]$ModuleName = $_.ModuleName
                     [string]$Size = $_.Size
                     [string]$FileName = $_.FileName
                     [string]$FileVersion = $_.FileVersion
                     Write-Host `"$server`"`,`"$ModuleName`"`,`"$Size`"`,`"$FileName`"`,`"$FileVersion`"
                 }
                 Stop-Process -Id $_.Id
             }
         }
     }
     End { }
 
 function get-listFromAD{
     $strFilter = "(objectCategory=Computer)"
     $props = "cn"
     $dse = [ADSI]"LDAP://rootdse"
     $root = "LDAP://"+$dse.defaultNamingContext
     $ds = New-Object DirectoryServices.DirectorySearcher([ADSI]$root,$strFilter,$props)
     $ds.PageSize = 1000
     $results = $ds.FindAll()
     foreach($obj in $results){
         $obj.cn
     }
 }
 
 
 if($args){ 
     foreach($i in $args){ 
         if($i -eq "ADquery"){ get-ModuleInfo -server get-listFromAD }
         else{ get-ModuleInfo -server $i }
     }
 }
 else{
     get-ModuleInfo
 }
`

