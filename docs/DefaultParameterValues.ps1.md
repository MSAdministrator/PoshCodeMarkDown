---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 3739
Published Date: 2012-11-02t15
Archived Date: 2012-12-24t00
---

# defaultparametervalues - 

## Description

for powershell 3

## Comments



## Usage



## TODO



## module

`export-defaultparameter`

## Code

`#
 #
 
 function Export-DefaultParameter {
 #.Synopsis
 [CmdletBinding()]
 param(
    [String]$Path = $(Join-Path (Split-Path $Profile.CurrentUserAllHosts -Parent) DefaultParameterValues.clixml)
 )
    Export-CliXml -InputObject $Global:PSDefaultParameterValues -Path $Path
 }
 
 function Import-DefaultParameter {
 #.Synopsis
 [CmdletBinding()]
 param(
    [String]$Path = $(Join-Path (Split-Path $Profile.CurrentUserAllHosts -Parent) DefaultParameterValues.clixml),
    [Switch]$Force
 )
    $TempParameterValues = $Global:PSDefaultParameterValues
    if(Test-Path $Path) {
       [System.Management.Automation.DefaultParameterDictionary]$NewValues = 
             Import-CliXml  -Path ${ProfileDir}\DefaultParameterValues.clixml
       
       $repeats = @()
       foreach($key in $NewValues.Keys) {
          if($key -eq "disabled") { continue }
          if($Global:PSDefaultParameterValues.ContainsKey($key)) {
             $command,$parameter = $key -split ":"
             if($Force) {
                $repeats += [PSCustomObject]@{Command=$Command;Parameter=$Parameter;NewDefault=$NewValues.$key;OldDefault=$Global:PSDefaultParameterValues.$Key}
                $Global:PSDefaultParameterValues.$Key = $NewValues.$key
             } else {
                $repeats += [PSCustomObject]@{Command=$Command;Parameter=$Parameter;CurrentDefault=$Global:PSDefaultParameterValues.$Key;SkippedValue=$NewValues.$key}
             }
          } else {
             $Global:PSDefaultParameterValues.$Key = $NewValues.$key
          }
       }
       if($repeats.Count) {
          $Width = &{
             $Local:ErrorActionPreference = "SilentlyContinue"
             $Width = $Host.UI.RawUI.BufferSize.Width - 2
             if($Width) { $Width } else { 80 }
          }
          if($Force) {
             Write-Warning ("Some defaults overwritten:`n{0}" -f ($repeats | Out-String -Width $Width))
          } else {
             Write-Warning ("Some defaults already defined, use -Force to overwrite:`n{0}" -f ($repeats | Out-String -Width $Width))
          }
       }
       $Global:PSDefaultParameterValues["Disabled"] = $false
    } else {
       Write-Warning "Default Parameter file not found: '$Path'"
    }
 }
 
 function Set-DefaultParameter {
 #.Synopsis
 #.Description
 [CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact="Low")]
 param(
    [parameter(Position=0,Mandatory=$true)]
    $Command,
    [parameter(Position=1,Mandatory=$true)]
    [String]$Parameter,
    [parameter(Position=2,Mandatory=$true)]
    $Value   
 )
     Write-Verbose ($PSBoundParameters | Out-String)
 
    if($parameter.StartsWith("-")) {
       $parameter = $parameter.TrimStart("-")
    }
  
    $cmd = Get-Command $command | Select -First 1
    while($cmd.CommandType -eq "Alias") {
       $cmd = Get-Command $cmd.Definition | Select -First 1
    }
    $parameters = $cmd.Parameters.Values
    $param = @($parameters | Where-Object { $_.Name -match $parameter -or $_.Aliases -match $parameter} | Select -Expand Name -Unique)
    if(!$param) {
       $param = @($parameters | Where-Object { $_.Name -match "${parameter}*"} | Select -First 1 -Expand Name)
       if(!$param) {
          $param = @($parameters | Where-Object { $_.Aliases -match "${parameter}*"} | Select -First 1 -Expand Name)
       }
    }
    if($param.Count -eq 1) {
       $Command = $Cmd.Name
       Write-Verbose "Setting Parameter $parameter on ${Command} to default: $($param[0])"
       
       $Key = "${Command}:$($param[0])"
       if( $Global:PSDefaultParameterValues.ContainsKey($Key) ) {
          $WI = "Overwrote the default parameter value for '$Key' with $Value"
          $CF = "Overwrite the existing default parameter value for '$Key'? ($($Global:PSDefaultParameterValues.$Key))"
       } else {
          $WI = "Added a default parameter value for '$Key' = $Value"
          $CF = "Add a default parameter value for '$Key'? ($($Global:PSDefaultParameterValues.$Key))"
       }
       if($PSCmdlet.ShouldProcess( $WI, $CI, "Setting Default Parameter Values" )) {
          $Global:PSDefaultParameterValues.$Key = $Value
       }
    } else {
       Write-Warning "Parameter $parameter not found on ${Command}"
    }
 }
 
 function Get-DefaultParameter {
 #.Synopsis
 #.Description
 [CmdletBinding()]
 param(
    [parameter(Position=0,Mandatory=$false)]
    [string]$Command = "*",
    [parameter(Position=1,Mandatory=$true)]
    [string]$Parameter = "*"
 )
 
    if($parameter.StartsWith("-")) {
       $parameter = $parameter.TrimStart("-")
    }
  
    $repeats = @()
    foreach($key in $Global:PSDefaultParameterValues.Keys) {
       if($key -eq "disabled" -and $Global:PSDefaultParameterValues["Disabled"]) { 
          Write-Warning "Default Parameter Values are DISABLED and will not be used:"
          continue
       }
       if($key -like "${Command}:${Parameter}*") {
          $Cmd,$Param = $key -split ":"
          $repeats += [PSCustomObject]@{Command=$Cmd;Parameter=$Param;CurrentDefault=$Global:PSDefaultParameterValues.$Key}
       } else {
          Write-Verbose "'$key' is -not -like '${Command}:${Parameter}*'"
       }
    }
    $repeats
 }
 
 
 function Remove-DefaultParameter {
 #.Synopsis
 #.Description
 [CmdletBinding(SupportsShouldProcess=$true, ConfirmImpact="Low")]
 param(
    [parameter(Position=0,Mandatory=$false)]
    [string]$Command = "*",
    [parameter(Position=1,Mandatory=$true)]
    [string]$Parameter
 )
 
 
    if($parameter.StartsWith("-")) {
       $parameter = $parameter.TrimStart("-")
    }
  
    $keys
    foreach($key in $Global:PSDefaultParameterValues.Keys | Where { $_ -like "${Command}:${Parameter}*" }) {
       if($key -ne "disabled") {
          if($PSCmdlet.ShouldProcess( "Removed the default parameter value for '$Key'",
                                      "Remove the default parameter value for '$Key'?",
                                      "Removing Default Parameter Values" )) {
             $Global:PSDefaultParameterValues.Remove($key)
          }
       }
    }
 }
 
 
 
 function Disable-DefaultParameter {
 #.Synopsis
 #.Description
 [CmdletBinding()]
 param()
    $Global:PSDefaultParameterValues["Disabled"]=$true
 }
 
 function Enable-DefaultParameter {
 #.Synopsis
 #.Description
 [CmdletBinding()]
 param()
    $Global:PSDefaultParameterValues["Disabled"]=$false
 }
 
 Set-Alias Add-DefaultParameter Set-DefaultParameter -ErrorAction SilentlyContinue
 Set-Alias sdp Set-DefaultParameter -ErrorAction SilentlyContinue
 Set-Alias gdp Get-DefaultParameter -ErrorAction SilentlyContinue
 Set-Alias rmdp Remove-DefaultParameter -ErrorAction SilentlyContinue
 Set-Alias ipdp Import-DefaultParameter -ErrorAction SilentlyContinue
 Set-Alias exdp Export-DefaultParameter -ErrorAction SilentlyContinue
 Import-DefaultParameter
 
 Export-ModuleMember -Alias * -Function Set-DefaultParameter, Get-DefaultParameter, Remove-DefaultParameter, `
                                        Import-DefaultParameter, Export-DefaultParameter, `
                                        Enable-DefaultParameter, Disable-DefaultParameter
`

