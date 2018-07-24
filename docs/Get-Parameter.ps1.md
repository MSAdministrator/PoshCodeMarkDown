---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.9
Encoding: ascii
License: cc0
PoshCode ID: 5929
Published Date: 2016-07-14t01
Archived Date: 2016-08-26t03
---

# get-parameter - 

## Description

this update adds a -setname filter and fixes bugs with ps3 and ps4 (also adds a “*” marker on the shortest form if it’s not an actual alias)

## Comments



## Usage



## TODO



## script

`join-object`

## Code

`#
 #
 
 #.Synopsis
 #.Notes
 #.Description
 #
 #.Example
 #.Example
 [CmdletBinding(DefaultParameterSetName="ParameterName")]
 param ( 
    [Parameter(Position = 1, Mandatory = $true, ValueFromPipelineByPropertyName = $true)]
    [Alias("Name")]
    [string[]]$CommandName,
 
    [Parameter(Position = 2, ValueFromPipelineByPropertyName=$true, ParameterSetName="FilterNames")]
    [string[]]$ParameterName = "*",
 
    [Parameter(ValueFromPipelineByPropertyName=$true, ParameterSetName="FilterSets")]
    [string[]]$SetName = "*",
 
    [Parameter(ValueFromPipelineByPropertyName = $true)]
    $ModuleName,
 
    [Switch]$SkipProviderParameters,
 
    [switch]$Force
 )
 
 Function global:Get-Parameter {
    #.Synopsis 
    #.Description
    #
    #.Example
    #.Example
    [CmdletBinding(DefaultParameterSetName="ParameterName")]
    param(
       [Parameter(Position = 1, Mandatory = $true, ValueFromPipelineByPropertyName = $true)]
       [Alias("Name")]
       [string[]]$CommandName,
 
       [Parameter(Position = 2, ValueFromPipelineByPropertyName=$true, ParameterSetName="FilterNames")]
       [string[]]$ParameterName = "*",
 
       [Parameter(ValueFromPipelineByPropertyName=$true, ParameterSetName="FilterSets")]
       [string[]]$SetName = "*",
 
       [Parameter(ValueFromPipelineByPropertyName = $true)]
       $ModuleName,
 
       [Switch]$SkipProviderParameters,
 
       [switch]$Force
    )
 
    begin {
       $PropertySet = @( "Name",
          @{n="Position";e={if($_.Position -lt 0){"Named"}else{$_.Position}}},
          "Aliases", 
          @{n="Short";e={$_.Name}},
          @{n="Type";e={$_.ParameterType.Name}}, 
          @{n="ParameterSet";e={$paramset}},
          @{n="Command";e={$command}},
          @{n="Mandatory";e={$_.IsMandatory}},
          @{n="Provider";e={$_.DynamicProvider}},
          @{n="ValueFromPipeline";e={$_.ValueFromPipeline}},
          @{n="ValueFromPipelineByPropertyName";e={$_.ValueFromPipelineByPropertyName}}
       )
       function Join-Object {
          Param(
            [Parameter(Position=0)]
            $First,
 
            [Parameter(ValueFromPipeline=$true,Position=1)]
            $Second
          )
          begin {
            [string[]] $p1 = $First | Get-Member -MemberType Properties | Select-Object -ExpandProperty Name
          }
          process {
            $Output = $First | Select-Object $p1
            foreach ($p in $Second | Get-Member -MemberType Properties | Where-Object {$p1 -notcontains $_.Name} | Select-Object -ExpandProperty Name) {
               Add-Member -InputObject $Output -MemberType NoteProperty -Name $p -Value $Second."$p"
            }
            $Output
          }
       }
 
       function Add-Parameters {
          [CmdletBinding()]
          param(
             [Parameter(Position=0)]
             [Hashtable]$Parameters,
 
             [Parameter(Position=1)]
             [System.Management.Automation.ParameterMetadata[]]$MoreParameters
          )
 
          foreach ($p in $MoreParameters | Where-Object { !$Parameters.ContainsKey($_.Name) } ) {
             Write-Debug ("INITIALLY: " + $p.Name)
             $Parameters.($p.Name) = $p | Select *
          }
 
          [Array]$Dynamic = $MoreParameters | Where-Object { $_.IsDynamic }
          if ($dynamic) {
             foreach ($d in $dynamic) {
                if (Get-Member -InputObject $Parameters.($d.Name) -Name DynamicProvider) {
                   Write-Debug ("ADD:" + $d.Name + " " + $provider.Name)
                   $Parameters.($d.Name).DynamicProvider += $provider.Name
                } else {
                   Write-Debug ("CREATE:" + $d.Name + " " + $provider.Name)
                   $Parameters.($d.Name) = $Parameters.($d.Name) | Select *, @{ n="DynamicProvider";e={ @($provider.Name) } }
                }
             } 
          }
       }
    }
 
    process {
       foreach ($cmd in $CommandName) {
          if ($ModuleName) {$cmd = "$ModuleName\$cmd"}
          Write-Verbose "Searching for $cmd"
          $commands = @(Get-Command $cmd)
 
          foreach ($command in $commands) {
             Write-Verbose "Searching for $command"
             while ($command.CommandType -eq "Alias") {
               $command = @(Get-Command ($command.definition))[0]
             }
             if (-not $command) {continue}
 
             Write-Verbose "Get-Parameters for $($Command.Source)\$($Command.Name)"
 
             $Parameters = @{}
 
             $NoProviderParameters = !$SkipProviderParameters
             if(!$SkipProviderParameters -and $Command.Source -eq "Microsoft.PowerShell.Management") {
                foreach($param in $Command.Parameters.Values) {
                   if(([String[]],[String] -contains $param.ParameterType) -and ($param.ParameterSets.Values | Where { $_.Position -ge 0 })) {
                      $NoProviderParameters = $false
                      break
                   }
                }
             }
 
             if($NoProviderParameters) {
                if($Command.Parameters) {
                   Add-Parameters $Parameters $Command.Parameters.Values
                }
             } else {
                foreach ($provider in Get-PSProvider) {
                   if($provider.Drives.Length -gt 0) {
                      $drive = Get-Location -PSProvider $Provider.Name
                   } else {
                      $drive = "{0}\{1}::\" -f $provider.ModuleName, $provider.Name
                   }
                   Write-Verbose ("Get-Command $command -Args $drive | Select -Expand Parameters")
 
                   try {
                      $MoreParameters = (Get-Command $command -Args $drive).Parameters.Values
                   } catch {}
        
                   if($MoreParameters.Length -gt 0) {
                      Add-Parameters $Parameters $MoreParameters
                   }
                }
                if($Parameters.Length -eq 0) {
                   if($Command.Parameters) {
                      Add-Parameters $Parameters $Command.Parameters.Values
                   }
                }
             }
 
             $ParameterNames = $Parameters.Keys + $Aliases
             foreach ($p in $($Parameters.Keys)) {
                $short = "^"
                $aliases = @($p) + @($Parameters.$p.Aliases) | sort { $_.Length }
                $shortest = "^" + @($aliases)[0]
 
                foreach($name in $aliases) {
                   $short = "^"
                   foreach ($char in [char[]]$name) {         
                      $short += $char
                      $mCount = ($ParameterNames -match $short).Count
                      if ($mCount -eq 1 ) {
                         if($short.Length -lt $shortest.Length) {
                            $shortest = $short
                         }
                         break
                      }
                   }
                }
                if($shortest.Length -lt @($aliases)[0].Length +1){
                   $Parameters.$p = $Parameters.$p | Add-Member NoteProperty Aliases ($Parameters.$p.Aliases + @("$($shortest.SubString(1))*")) -Force -Passthru
                }
             }
 
             $CommonParameters = [string[]][System.Management.Automation.Cmdlet]::CommonParameters
 
             foreach ($paramset in @($command.ParameterSets | Select-Object -ExpandProperty "Name")) {
                $paramset = $paramset | Add-Member -Name IsDefault -MemberType NoteProperty -Value ($paramset -eq $command.DefaultParameterSet) -PassThru
                foreach ($parameter in $Parameters.Keys | Sort-Object) {
                   if (!$Force -and ($CommonParameters -contains $Parameter)) {continue}
                   if ($Parameters.$Parameter.ParameterSets.ContainsKey($paramset) -or $Parameters.$Parameter.ParameterSets.ContainsKey("__AllParameterSets")) {
                      if ($Parameters.$Parameter.ParameterSets.ContainsKey($paramset)) {
                         $output = Join-Object $Parameters.$Parameter $Parameters.$Parameter.ParameterSets.$paramSet 
                      } else {
                         $output = Join-Object $Parameters.$Parameter $Parameters.$Parameter.ParameterSets.__AllParameterSets
                      }
 
                      Write-Output $Output | Select-Object $PropertySet | ForEach-Object {
                            $null = $_.PSTypeNames.Insert(0,"System.Management.Automation.ParameterMetadata")
                            $null = $_.PSTypeNames.Insert(0,"System.Management.Automation.ParameterMetadataEx")
                            $_
                         } |
                         Add-Member ScriptMethod ToString { $this.Name } -Force -Passthru |
                         Where-Object {$(foreach($pn in $ParameterName) {$_ -like $Pn}) -contains $true} |
                         Where-Object {$(foreach($sn in $SetName) {$_.ParameterSet -like $sn}) -contains $true}
 
                   }
                }
             }
          }
       }
    }
 }
 
 
 $tempFile = "$([System.IO.Path]::GetTempPath())Get-Parameter.ps1xml"
 Set-Content $tempFile @'
 <?xml version="1.0" encoding="utf-8" ?>
 <Configuration>
    <Controls>
       <Control>
          <Name>ParameterGroupingFormat</Name>
           <CustomControl>
              <CustomEntries>
                <CustomEntry>
                   <CustomItem>
                      <Frame>
                        <LeftIndent>4</LeftIndent>
                        <CustomItem>
                           <Text>Command: </Text>
                           <ExpressionBinding>
                              <ScriptBlock>"{0}/{1}" -f $(if($_.command.ModuleName){$_.command.ModuleName}else{$_.Command.CommandType.ToString()+":"}),$_.command.Name</ScriptBlock>
                           </ExpressionBinding>
                           <NewLine/>
                           <Text>Set:    </Text>
                           <ExpressionBinding>
                              <ScriptBlock>"$(if($_.ParameterSet -eq "__AllParameterSets"){"Default"}else{$_.ParameterSet})" + "$(if($_.ParameterSet.IsDefault){" *"})"</ScriptBlock>
                           </ExpressionBinding>
                           <NewLine/>
                        </CustomItem> 
                      </Frame>
                   </CustomItem>
                </CustomEntry>
              </CustomEntries>
          </CustomControl>
       </Control>
    </Controls>
    <ViewDefinitions>
       <View>
          <Name>ParameterMetadataEx</Name>
          <ViewSelectedBy>
            <TypeName>System.Management.Automation.ParameterMetadataEx</TypeName>
          </ViewSelectedBy>
          <GroupBy>
            <PropertyName>ParameterSet</PropertyName>
            <CustomControlName>ParameterGroupingFormat</CustomControlName>  
          </GroupBy>
          <TableControl>
             <TableHeaders>
                <TableColumnHeader>
                   <Label>Name</Label>
                   <Width>22</Width>
                </TableColumnHeader>
                <TableColumnHeader>
                   <Label>Aliases</Label>
                   <Width>12</Width>
                </TableColumnHeader>
                <TableColumnHeader>
                   <Label>Position</Label>
                   <Width>8</Width>
                </TableColumnHeader>
                <TableColumnHeader>
                   <Label>Mandatory</Label>
                   <Width>9</Width>
                </TableColumnHeader>
                <TableColumnHeader>
                   <Label>Pipeline</Label>
                   <Width>8</Width>
                </TableColumnHeader>
                <TableColumnHeader>
                   <Label>ByName</Label>
                   <Width>6</Width>
                </TableColumnHeader>
                <TableColumnHeader>
                   <Label>Provider</Label>
                   <Width>15</Width>
                </TableColumnHeader>
                <TableColumnHeader>
                   <Label>Type</Label>
                </TableColumnHeader>
             </TableHeaders>
             <TableRowEntries>
                <TableRowEntry>
                   <TableColumnItems>
                      <TableColumnItem>
                         <PropertyName>Name</PropertyName>
                      </TableColumnItem>
                      <TableColumnItem>
                         <PropertyName>Aliases</PropertyName>
                      </TableColumnItem>
                      <TableColumnItem>
                         <!--PropertyName>Position</PropertyName-->
                         <ScriptBlock>if($_.Position -lt 0){"Named"}else{$_.Position}</ScriptBlock>
                      </TableColumnItem>
                      <TableColumnItem>
                         <PropertyName>Mandatory</PropertyName>
                      </TableColumnItem>
                      <TableColumnItem>
                         <PropertyName>ValueFromPipeline</PropertyName>
                      </TableColumnItem>
                      <TableColumnItem>
                         <PropertyName>ValueFromPipelineByPropertyName</PropertyName>
                      </TableColumnItem>
                      <TableColumnItem>
                         <!--PropertyName>Provider</PropertyName-->
                         <ScriptBlock>if($_.Provider){$_.Provider}else{"All"}</ScriptBlock>
                      </TableColumnItem>
                      <TableColumnItem>
                         <PropertyName>Type</PropertyName>
                      </TableColumnItem>
                   </TableColumnItems>
                </TableRowEntry>
             </TableRowEntries>
          </TableControl>
       </View>
    </ViewDefinitions>
 </Configuration>
 '@
 
 Update-FormatData -Append $tempFile
 
 Get-Parameter @PSBoundParameters
`

