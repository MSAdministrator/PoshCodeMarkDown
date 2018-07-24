---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6241
Published Date: 2016-02-29t14
Archived Date: 2016-05-02t20
---

# restart-iisapppool - 

## Description

restarts local or remote iis apppools

## Comments



## Usage



## TODO



## function

`restart-iisapppool`

## Code

`#
 #
 function Restart-IISAppPool {
    [CmdletBinding(SupportsShouldProcess=$true)]
    #.Synopsis
    #.Parameter ComputerName
    #.Parameter AppPool
    #.Parameter Credential
    #.Example
    #
    #.Example
    #
    param(
       [Parameter(ValueFromPipelineByPropertyName=$true, Mandatory= $true, Position=0)]
       [Alias("Name","Pool")]
       [String[]]$AppPool
    ,
       [Parameter(ValueFromPipelineByPropertyName=$true, Position=1)]
       [Alias("CN","Server","__Server")]
       [String]$ComputerName
    ,
       [Parameter()]
       [System.Management.Automation.Credential()]
       $Credential = [System.Management.Automation.PSCredential]::Empty
    )
    
    begin {
       if ($Credential -and ($Credential -ne [System.Management.Automation.PSCredential]::Empty)) {
          $Credential = $Credential.GetNetworkCredential()
       }
       Write-Debug ("BEGIN:`n{0}" -f ($PSBoundParameters | Out-String))
 
       $Skip = $false
       if($PSBoundParameters.ContainsKey("AppPool") -and $AppPool -match "\*") {
         $null = $PSBoundParameters.Remove("AppPool")
         $null = $PSBoundParameters.Remove("WhatIf")
         $null = $PSBoundParameters.Remove("Confirm")
         Write-Verbose "Searching for AppPools matching: $($AppPool -join ', ')"
 
         Get-WmiObject IISApplicationPool -Namespace root\MicrosoftIISv2 -Authentication PacketPrivacy @PSBoundParameters | 
         Where-Object { @(foreach($pool in $AppPool){ $_.Name -like $Pool -or $_.Name -like "W3SVC/APPPOOLS/$Pool" }) -contains $true } |
         Restart-IISAppPool
         $Skip = $true
       }
       $ProcessNone = $ProcessAll = $false;
    }
    process {
       Write-Debug ("PROCESS:`n{0}" -f ($PSBoundParameters | Out-String))
    
       if(!$Skip) {
         $null = $PSBoundParameters.Remove("AppPool")
         $null = $PSBoundParameters.Remove("WhatIf")
         $null = $PSBoundParameters.Remove("Confirm")
          foreach($pool in $AppPool) {
             Write-Verbose "Processing $Pool"
             if($PSCmdlet.ShouldProcess("Would restart the AppPool '$Pool' on the '$(if($ComputerName){$ComputerName}else{'local'})' server.", "Restart ${Pool}?", "Restarting IIS App Pools on $ComputerName")) {
                Write-Verbose "Restarting $Pool"
                  Invoke-WMIMethod Recycle -Path "IISApplicationPool.Name='$Pool'" -Namespace root\MicrosoftIISv2 -Authentication PacketPrivacy @PSBoundParameters
             }
          }
       }
    }
 }
`

