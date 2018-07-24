---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 2460
Published Date: 2011-01-17t08
Archived Date: 2011-11-12t15
---

# new-odataserviceproxy - 

## Description

a wrapper for datasvcutil to generate web service proxies in-memory for odata services like netflix (which are not handled correctly by powershell’s built-in new-webserviceproxy).

## Comments



## Usage



## TODO



## function

`new-odataserviceproxy`

## Code

`#
 #
 function New-ODataServiceProxy {
 #.Synopsis
 #.Description 
 #.Parameter URI
 #.Parameter Name
 #.Parameter Passthru
 #.Example
 #.Example
 #
 #.Example
 #
 #.Link 
 #.Notes
 #
 param(
 [Parameter(Mandatory=$true)]
 [Alias("Uri","WSU")]
 [String]$WebServiceUri = "http://odata.netflix.com/Catalog/", 
 [Alias("Name","VN")]
 [String]$VariableName = "ODataServiceProxy",
 [switch]$Passthru
 )
 
 if(!(Test-Path Variable::Global:ODataServices)){
    [Reflection.Assembly]::Load("System.Data.Services.Client, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089") | Out-Null
    $global:ODataServices = @{}
 }
 
 $normalized = ([uri]$WebServiceUri).AbsoluteUri.TrimEnd("/") 
 
 if($global:ODataServices.ContainsKey($normalized)) {
    New-Object $global:ODataServices.$normalized.ContextType $WebServiceUri
    if($Passthru) {
       $global:ODataServices.$normalized.OtherTypes
    }
    return
 }
 
 switch($PSVersionTable.ClrVersion.Major) {
       Set-Alias Get-ODataServiceDefinition (Get-ChildItem ([System.Runtime.InteropServices.RuntimeEnvironment]::GetRuntimeDirectory())  DataSvcUtil.exe)
       break
    }
       $FrameworkPath = [System.Runtime.InteropServices.RuntimeEnvironment]::GetRuntimeDirectory() | Split-Path
       Set-Alias Get-ODataServiceDefinition (Get-ChildItem $FrameworkPath\v3.5\DataSvcUtil.exe)
       break
    }
    default { throw "This script is out of date, please fix it and upload a new one to PoshCode!" }   
 }
 $temp = [IO.Path]::GetTempFileName()
 Get-ODataServiceDefinition -out:$temp -uri:$WebServiceUri -nologo | tee -var output | out-null
 if(!$?) {
    Write-Error ($output -join "`n")
    return
 }
 Remove-Item $temp
 
 switch($PSVersionTable.ClrVersion.Major) {
    4 { 
          $Types = Add-Type $code -Reference "System.Data.Services.Client, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089", "System.Core" -Passthru
    }
    2 {
          $Types = Add-Type $code -Reference "System.Data.Services.Client, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" -Passthru -Language CSharpVersion3 
    }
    default { throw "This script is out of date, please fix it and upload a new one to PoshCode!" }   
 }
 
 if(!$Types) { return }
 
 $ContextType = $Types | Where-Object { $_.BaseType -eq [System.Data.Services.Client.DataServiceContext] }
 $global:ODataServices.$normalized = New-Object PSObject -Property @{ContextType=$ContextType; OtherTypes=$([Type[]]($Types|Where-Object{$_.BaseType -ne [System.Data.Services.Client.DataServiceContext]}))}
 $ctx = new-object $ContextType $WebServiceUri
 
 $VariableName = $VariableName.Split(":")[-1]
 if(Test-Path Variable:$VariableName) { Remove-Variable $VariableName -Force }
 Set-Variable -Name "Global:$VariableName" -Value $ctx -Description "An OData WebService Proxy to $WebServiceUri" -Option ReadOnly, AllScope -Passthru
 if($Passthru) { Write-Output $types }
 }
`

