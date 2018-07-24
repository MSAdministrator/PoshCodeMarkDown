---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 6489
Published Date: 2016-08-23t08
Archived Date: 2016-10-27t09
---

# home automation module - 

## Description

this is an updated version of the home automation module. the main difference is that some output is removed (no news is good news…) if “-verbose” is not used and that it has a separate cmdlet for connecting to telldus live! (connect-tellduslive) which uses a pscredential.

## Comments



## Usage



## TODO



## function

`connect-tellduslive`

## Code

`#
 #
 #========================================================================
 #========================================================================
  
  
 function Connect-TelldusLive
 {
     [cmdletbinding()]
     param(
           [Parameter(Mandatory=$True)]
           [System.Management.Automation.PSCredential] $Credential)
 
 
     $LoginPostURI="https://login.telldus.com/openid/server?openid.ns=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0&openid.mode=checkid_setup&openid.return_to=http%3A%2F%2Fapi.telldus.com%2Fexplore%2Fclients%2Flist&openid.realm=http%3A%2F%2Fapi.telldus.com&openid.ns.sreg=http%3A%2F%2Fopenid.net%2Fextensions%2Fsreg%2F1.1&openid.sreg.required=email&openid.claimed_id=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0%2Fidentifier_select&openid.identity=http%3A%2F%2Fspecs.openid.net%2Fauth%2F2.0%2Fidentifier_select#"
     $turnOffURI="https://api.telldus.com/explore/device/turnOff"
 
     $TelldusWEB = Invoke-WebRequest $turnOffURI -SessionVariable Global:Telldus
 
     $form = $TelldusWEB.Forms[0]
     $form.Fields["email"] = $Credential.UserName
     $form.Fields["password"] = $Credential.GetNetworkCredential().Password
 
     $TelldusWEB = Invoke-WebRequest -Uri $LoginPostURI -WebSession $Global:Telldus -Method POST -Body $form.Fields
 
     $form = $null
 
     [gc]::Collect()
 }
 
 function Get-TDDevice
 {
     <#
     .SYNOPSIS
     Retrieves all devices associated with a Telldus Live! account.
 
     .DESCRIPTION
     This command will list all devices associated with an Telldus Live!-account and their current status and other information.
 
     .EXAMPLE
     Get-TDDevice
 
     .EXAMPLE
     Get-TDDevice | Format-Table
 
     #>
 
     if ($Telldus -eq $null) {
         Write-Error "You must first connect using the Connect-TelldusLive cmdlet"
         return
     }
 
     $PostActionURI="https://api.telldus.com/explore/doCall"
     $Action='list'
     $SupportedMethods=19
 
     $request = @{'group'='devices';'method'= $Action;'param[supportedMethods]'= $SupportedMethods;'responseAsXml'='xml'}
 
     [xml] $ActionResults=Invoke-WebRequest -Uri $PostActionURI -WebSession $Global:Telldus -Method POST -Body $request
 
     $Results=$ActionResults.devices.ChildNodes
 
     foreach ($Result in $Results)
     {
         $PropertiesToOutput = @{
                              'Name' = $Result.name;
                              'State' = switch ($Result.state)
                                        {
                                              1 { "On" }
                                              2 { "Off" }
                                             16 { "Dimmed" }
                                             default { "Unknown" }
                                        }
                              'DeviceID' = $Result.id;
                              
 
                              'Statevalue' = $Result.statevalue
                              'Methods' = switch ($Result.methods)
                                          {
                                              3 { "On/Off" }
                                             19 { "On/Off/Dim" }
                                             default { "Unknown" }
                                          }
                              'Type' = $Result.type;
                              'Client' = $Result.client;
                              'ClientName' = $Result.clientName;
                              'Online' = switch ($Result.online)
                                         {
                                             0 { $false }
                                             1 { $true }
                                         }
                              }
 
         $returnObject = New-Object -TypeName PSObject -Property $PropertiesToOutput
 
         Write-Output $returnObject | Select-Object Name, DeviceID, State, Statevalue, Methods, Type, ClientName, Client, Online
     }
 }
 
 function Set-TDDevice
 {
 
     <#
     .SYNOPSIS
     Turns a device on or off.
 
     .DESCRIPTION
     This command can set the state of a device to on or off through the Telldus Live! service.
 
     .EXAMPLE
     Set-TDDevice -DeviceID 123456 -Action turnOff
 
     .EXAMPLE
     Set-TDDevice -DeviceID 123456 -Action turnOn
 
     .PARAMETER DeviceID
     The DeviceID of the device to turn off or on. (Pipelining possible)
 
     .PARAMETER Action
     What to do with that device. Possible values are "turnOff" or "turnOn".
 
     #>
 
     [CmdletBinding()]
     param(
 
       [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
       [Alias('id')]
       [string] $DeviceID,
       [Parameter(Mandatory=$True)]
       [ValidateSet("turnOff","turnOn")]
       [string] $Action)
 
 
     BEGIN {
         if ($Telldus -eq $null) {
             Write-Error "You must first connect using the Connect-TelldusLive cmdlet"
             return
         }
 
         $PostActionURI = "https://api.telldus.com/explore/doCall"
     }
 
     PROCESS {
 
         $request = @{'group'='device';'method'= $Action;'param[id]'= $DeviceID;'responseAsXml'='xml'}
 
         [xml] $ActionResults=Invoke-WebRequest -Uri $PostActionURI -WebSession $Global:Telldus -Method POST -Body $request
 
         $Results=$ActionResults.device.status -replace "\s"
 
         Write-Verbose "Doing action $Action on device $DeviceID. Result: $Results."
     }
 }
 
 function Get-TDSensor
 {
     <#
     .SYNOPSIS
     Retrieves all sensors associated with a Telldus Live! account.
 
     .DESCRIPTION
     This command will list all sensors associated with an Telldus Live!-account and their current status and other information.
 
     .EXAMPLE
     Get-TDSensor
 
     .EXAMPLE
     Get-TDSensor | Format-Table
 
     #>
 
     if ($Telldus -eq $null) {
         Write-Error "You must first connect using the Connect-TelldusLive cmdlet"
         return
     }
 
     $PostActionURI="https://api.telldus.com/explore/doCall"
 
     $SensorListKeys = @{
         'group' = 'sensors'
         'method' = 'list'
         'field_includeIgnored' = ''
         'field_includeValues' = ''
         'field_includeScale' = ''
         'field_useAlternativeData' = ''
         'responseAsXml' = 'xml'
         'responseAsJson' = 'json'
     }
 
     $ActionResults=$null
 
     [xml] $ActionResults=Invoke-WebRequest -Uri $PostActionURI -WebSession $Global:Telldus -Method POST -Body $SensorListKeys
     [datetime] $TelldusDate="1970-01-01 00:00:00"
 
     $TheResults=$ActionResults.sensors.ChildNodes
 
     foreach ($Result in $TheResults) {
         $SensorInfo=$Result
 
         $DeviceID=$SensorInfo.id.trim()
         $SensorName=$SensorInfo.name.trim()
         $SensorLastUpdated=$SensorInfo.lastupdated.trim()
         $SensorLastUpdatedDate=$TelldusDate.AddSeconds($SensorLastUpdated)
         $clientid=$SensorInfo.client.trim()
         $clientName=$SensorInfo.clientname.trim()
         $sensoronline=$SensorInfo.online.trim()
 
         $returnObject = New-Object System.Object
         $returnObject | Add-Member -Type NoteProperty -Name DeviceID -Value $DeviceID
         $returnObject | Add-Member -Type NoteProperty -Name Name -Value $SensorName
         $returnObject | Add-Member -Type NoteProperty -Name LocationID -Value $clientid
         $returnObject | Add-Member -Type NoteProperty -Name LocationName -Value $clientName
         $returnObject | Add-Member -Type NoteProperty -Name LastUpdate -Value $SensorLastUpdatedDate
         $returnObject | Add-Member -Type NoteProperty -Name Online -Value $sensoronline
 
         Write-Output $returnObject
     }
 }
 
 function Get-TDSensorData
 {
     <#
     .SYNOPSIS
     Retrieves the sensordata of specified sensor.
 
     .DESCRIPTION
     This command will retrieve the sensordata associated with the specified ID.
 
     .EXAMPLE
     Get-TDSensorData -DeviceID 123456
 
     .PARAMETER DeviceID
     The DeviceID of the sensor which data you want to retrieve.
 
     #>
 
     [CmdletBinding()]
     param(
 
       [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)] [Alias('id')] [string] $DeviceID)
 
     BEGIN {
         if ($Telldus -eq $null) {
             Write-Error "You must first connect using the Connect-TelldusLive cmdlet"
             return
         }
 
         $sensorDataURI="https://api.telldus.com/explore/sensor/info"
         $PostActionURI="https://api.telldus.com/explore/doCall"
     }
 
     PROCESS {
         $request = @{'group'='sensor';'method'= 'info';'param[id]'= $DeviceID;'responseAsXml'='xml'}
 
         [xml] $ActionResults=Invoke-WebRequest -Uri $PostActionURI -WebSession $Global:Telldus -Method POST -Body $request
         [datetime] $TelldusDate="1970-01-01 00:00:00"
 
         $SensorInfo=$ActionResults.sensor
         $SensorData=$ActionResults.sensor.data
 
         $SensorName=$SensorInfo.name.trim()
         $SensorLastUpdated=$SensorInfo.lastupdated.trim()
         $SensorLastUpdatedDate=$TelldusDate.AddSeconds($SensorLastUpdated)
         $clientName=$SensorInfo.clientname.trim()
         $SensorTemp=($SensorData | ? name -eq "temp").value | select -First 1
         $SensorHumidity=($SensorData | ? name -eq "humidity").value | select -First 1
 
         $returnObject = New-Object System.Object
         $returnObject | Add-Member -Type NoteProperty -Name DeviceID -Value $DeviceID
         $returnObject | Add-Member -Type NoteProperty -Name Name -Value $SensorName
         $returnObject | Add-Member -Type NoteProperty -Name LocationName -Value $clientName
         $returnObject | Add-Member -Type NoteProperty -Name Temperature -Value $SensorTemp
         $returnObject | Add-Member -Type NoteProperty -Name Humidity -Value $SensorHumidity
         $returnObject | Add-Member -Type NoteProperty -Name LastUpdate -Value $SensorLastUpdatedDate
 
         Write-Output $returnObject
     }
 }
 
 function Set-TDDimmer
 {
     <#
     .SYNOPSIS
     Dims a device to a certain level.
 
     .DESCRIPTION
     This command can set the dimming level of a device to through the Telldus Live! service.
 
     .EXAMPLE
     Set-TDDimmer -DeviceID 123456 -Level 89
 
     .EXAMPLE
     Set-TDDimmer -Level 180
 
     .PARAMETER DeviceID
     The DeviceID of the device to dim. (Pipelining possible)
 
     .PARAMETER Level
     What level to dim to. Possible values are 0 - 255.
 
     #>
 
     [CmdletBinding()]
     param(
 
       [Parameter(Mandatory=$True,
                  ValueFromPipeline=$true,
                  ValueFromPipelineByPropertyName=$true,
                  HelpMessage="Enter the DeviceID.")] [Alias('id')] [string] $DeviceID,
 
       [Parameter(Mandatory=$True,
                  HelpMessage="Enter the level to dim to between 0 and 255.")]
       [ValidateRange(0,255)]
       [int] $Level)
 
 
     BEGIN {
 
         if ($Telldus -eq $null) {
             Write-Error "You must first connect using the Connect-TelldusLive cmdlet"
             return
         }
 
         $PostActionURI="https://api.telldus.com/explore/doCall"
         $Action='dim'
     }
 
     PROCESS {
 
         $request = @{'group'='device';'method'= $Action;'param[id]'= $DeviceID;'param[level]'= $Level;'responseAsXml'='xml'}
 
         [xml] $ActionResults=Invoke-WebRequest -Uri $PostActionURI -WebSession $Global:Telldus -Method POST -Body $request
 
         $Results=$ActionResults.device.status -replace "\s"
 
         Write-Verbose "Dimming device $DeviceID to level $Level. Result: $Results."
     }
 }
 
 function Get-TDDeviceHistory
 {
     <#
     .SYNOPSIS
     Retrieves all events associated with the specified device.
     .DESCRIPTION
     This command will list all events associated with the specified device
     .EXAMPLE
     Get-TDDeviceHistory
     .EXAMPLE
     Get-TDDeviceHistory | Format-Table
     #>
 
     [cmdletbinding()]
     param(
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [Alias('id')]
     [string] $DeviceID)
 
     BEGIN {
         if ($Telldus -eq $null) {
             Write-Error "You must first connect using the Connect-TelldusLive cmdlet"
             return
         }
     }
 
     PROCESS {
         $PostActionURI="https://live.telldus.com/device/history?id=$DeviceID"
 
         $HistoryEvents = Invoke-RestMethod -Uri $PostActionURI -WebSession $Global:Telldus | select -ExpandProperty History
 
         foreach ($HistoryEvent in $HistoryEvents)
         {
             $PropertiesToOutput = @{
                                  'DeviceID' = $DeviceID
                                  'State' = switch ($HistoryEvent.state)
                                            {
                                                  1 { "On" }
                                                  2 { "Off" }
                                                 16 { "Dimmed" }
                                                 default { "Unknown" }
                                            }
                                  'Statevalue' = $HistoryEvent.statevalue
                                  'Origin' = $HistoryEvent.Origin;
                                  'EventDate' = (Get-Date "1970-01-01 00:00:00").AddSeconds($HistoryEvent.ts)
                                  }
 
             $returnObject = New-Object -TypeName PSObject -Property $PropertiesToOutput
 
             Write-Output $returnObject | Select-Object DeviceID, EventDate, State, Statevalue, Origin
         }
     }
 
     END { }
 }
 
 
 function Get-TDSensorHistoryData
 {
     <#
     .SYNOPSIS
     Retrieves sensor data history from Telldus Live!
     
     .DESCRIPTION
     This command will retrieve the sensor history data of the specified sensor.
     
     .PARAMETER DeviceID
     The DeviceID of the sensor which data you want to retrieve.
 
     .PARAMETER After
     Specify from which date you would like to retrieve sensor history.
 
     .PARAMETER Before
     Specify the "end date" of the data samples.
     Default value is current date.
 
     .EXAMPLE
     Get-TDSensorHistoryData -DeviceID 123456
 
     .EXAMPLE
     Get-TDSensorHistoryData -DeviceID 123456 | Format-Table
 
     .EXAMPLE
     Get-TDSensorHistoryData -DeviceID 123456 -After (get-date).AddDays(-1)
     
     Get's the history from yesterday until today
 
     #>
 
     [cmdletbinding(DefaultParameterSetName='AllData')]
     param(
         [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName='AllData')]
         [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName='DateRange')]
         [Alias('id')]
         [string] $DeviceID,
 
         [Parameter(Mandatory=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName='DateRange')]
         [DateTime] $After,
 
         [Parameter(Mandatory=$false, ValueFromPipelineByPropertyName=$true, ParameterSetName='DateRange')]
         [DateTime] $Before
     )
 
     BEGIN {
         if ($Telldus -eq $null) {
             Write-Error "You must first connect using the Connect-TelldusLive cmdlet"
             return
         }
     }
 
     PROCESS {
         $PostActionURI="https://live.telldus.com/sensor/history?id=$DeviceID"
 
         if ($PSCmdlet.ParameterSetName -eq 'DateRange') {
             if (-not $Before) {
                 $Before = (Get-Date).ToUniversalTime()
             }
 
             if ($Before -gt $After) {
                 $FromDateToPost = [Math]::Floor((New-TimeSpan -Start '1970-01-01' -End $After.ToUniversalTime()).TotalSeconds)
                 $ToDateToPost = [Math]::Floor((New-TimeSpan -Start '1970-01-01' -End $Before.ToUniversalTime()).TotalSeconds)
 
                 $PostActionURI = $PostActionURI + "&fromdate=$FromDateToPost" + "&todate=$ToDateToPost"
             }
             else {
                 throw 'The value for Before must be greater than the value for After.'
             }
         }
 
         $HistoryDataPoints = Invoke-RestMethod -Uri $PostActionURI -WebSession $Global:Telldus | Select-Object -ExpandProperty History
 
         foreach ($HistoryDataPoint in $HistoryDataPoints)
         {
             $PropertiesToOutput = @{
                                  'DeviceID' = $DeviceID
                                  'Humidity' = ($HistoryDataPoint.data | Where-Object { $_.Name -eq 'humidity' }).value
                                  'Temperature' = ($HistoryDataPoint.data | Where-Object { $_.Name -eq 'temp' }).value
                                  'Date' = (Get-Date "1970-01-01 00:00:00").AddSeconds($HistoryDataPoint.ts)
                                  }
 
             $returnObject = New-Object -TypeName PSObject -Property $PropertiesToOutput
 
             Write-Output $returnObject | Select-Object DeviceID, Humidity, Temperature, Date
         }
     }
 
     END { }
 }
`

