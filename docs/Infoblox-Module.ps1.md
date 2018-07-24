---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 1.2.1/record
Encoding: ascii
License: cc0
PoshCode ID: 5044
Published Date: 2016-04-02t20
Archived Date: 2016-02-21t01
---

# infoblox module - 

## Description

i’ve just started to work on a powershell module for the infoblox trinzic ddi appliance, this is a very early release with some functions that can manage dns-records.

## Comments



## Usage



## TODO



## function

`get-ibresourcerecord`

## Code

`#
 #
 function Get-IBResourceRecord
 {
 
     <#
     .SYNOPSIS
     Retrieves resource records from a Infoblox Gridserver
 
     .DESCRIPTION
     Specify an attribute to search for, for example hostname and retrieve the object from the Gridserver
 
     .EXAMPLE
     Get-InfoBloxRecord -RecordType host -RecordName MyServer -GridServer myinfoblox.mydomain.com -Credential $Credential
 
     .EXAMPLE
     Get-InfoBloxRecord -RecordType network -RecordName 1.0.0.0/8 -GridServer myinfoblox.mydomain.com -Credential $Credential -Passthrough
 
     .PARAMETER RecordType
     Specify the type of record, for example host or network.
 
     .PARAMETER SearchField
     The field where the RecordValue is. Default is "Name".
 
     .PARAMETER RecordValue
     The value to search for.
 
     .PARAMETER GridServer
     The name of the infoblox appliance.
 
     .PARAMETER Properties
     What properties should be included?
 
     .PARAMETER Credential
     Add a Powershell credential object (created with for example Get-Credential).
 
     .PARAMETER Passthrough
     Includes credentials and gridserver in the object sent down the pipeline so you don't need to add them in the next cmdlet.
 
     #>
 
     [CmdletBinding()]
 
     param(
     [Parameter(Mandatory=$True)]
     [ValidateSet("A","AAAA","CName","DName","DNSKEY","DS","Host","LBDN","MX","NAPTR","NS","NSEC","NSEC3","NSEC3PARAM","PTR","RRSIG","SRV","TXT")]
     [string] $RecordType,
     [Parameter(Mandatory=$false)]
     $SearchField = 'name',
     [Parameter(Mandatory=$True)]
     $RecordValue,
     [Parameter(Mandatory=$True)]
     $GridServer,
     [Parameter(Mandatory=$false)]
     $Properties,
     [switch] $Passthrough,
     [Parameter(Mandatory=$True)]
     $Credential)
 
     BEGIN { }
 
     PROCESS {
 
         Write-Verbose "Building resource record search query..."
         $InfobloxURI = "https://$GridServer/wapi/v1.2.1/record:$($RecordType.ToLower())`?$($SearchField.ToLower())~=$RecordValue"
 
         if ($Properties -ne $null) {
             Write-Verbose "Adding return fields/properties..."
             $InfobloxURI = $InfobloxURI + "&_return_fields=$(($Properties -join "," -replace " ").ToLower())"
         }
 
         Write-Verbose "Initiating webrequest to API..."
 
         $WebRequest = Invoke-WebRequest -Uri $InfobloxURI -Credential $Credential
 
         Write-Verbose "Checking status code..."
 
         if ($WebRequest.StatusCode -eq 200) {
             Write-Verbose "Statuscode OK. Converting from Json..."
             $RecordObj = $WebRequest.Content | ConvertFrom-Json -ErrorAction Stop
         }
         else {
             Write-Error "Request to Infoblox failed (response code is not 200). See error above for details."
             Write-Debug "Request just failed (response not 200 or Json version failed). Please debug."
             return
         }
 
 
         Write-Verbose "Looping through returned objects..."
 
         foreach ($Record in $RecordObj) {
 
             $returnObject = $null
             $returnObject = $Record
 
             if ($Passthrough -eq $true) {
                 Write-Verbose "Adding credentials/gridserver to the pipeline..."
                 $returnObject | Add-Member -Type NoteProperty -Name Credential -Value $Credential
                 $returnObject | Add-Member -Type NoteProperty -Name GridServer -Value $GridServer
             }
 
             Write-Verbose "Sending object to the pipeline..."
             Write-Output $returnObject
         }
     }
 
     END {
         Write-Verbose "Finished."
     }
 
 }
 
 function Add-IBResourceRecordHost
 {
 
     <#
     .SYNOPSIS
     Add a host record on the Infoblox Gridserver
 
     .DESCRIPTION
     This cmdlet creates a host object on the Infoblox Gridserver.
 
     .EXAMPLE
     Add-InfoBloxHostRecord -IPv4Address 1.2.3.4 -HostName myserver.mydomain.com -GridServer myinfoblox.mydomain.com -Credential $Credential
 
     .PARAMETER IPv4Address
     The IPv4 address for the host record. Allows pipeline input.
 
     .PARAMETER HostName
     The hostname for the host record. Allows pipeline input.
 
     .PARAMETER GridServer
     The name of the infoblox appliance. Allows pipeline input.
 
     .PARAMETER Credential
     Add a Powershell credential object (created with for example Get-Credential). Allows pipeline input.
 
     #>
 
     [CmdletBinding()]
     param(
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     $IPv4Address,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [Alias('name')]
     $HostName,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [string] $GridServer,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [System.Management.Automation.PSCredential] $Credential)
 
     BEGIN { }
 
     PROCESS {
 
         $InfobloxURI = "https://$GridServer/wapi/v1.2.1/record:host"
 
         $Data = "{`"ipv4addrs`":[{`"ipv4addr`":'$IPv4Address'}],`"name`":'$HostName'}" | ConvertFrom-Json | ConvertTo-Json
 
         $WebReqeust = Invoke-WebRequest -Uri $InfobloxURI -Method Post -Body $Data -ContentType "application/json" -Credential $Credential
 
         if ($WebReqeust.StatusCode -ne 201) {
             Write-Error "Request to Infoblox failed for record $HostName!"
             return
         }
     }
 
     END { }
 }
 
 
 function Add-IBResourceRecordCName
 {
 
     <#
     .SYNOPSIS
     Adds a CNAME record on the Infoblox Gridserver
 
     .DESCRIPTION
     This cmdlet creates a CName record on the Infoblox Gridserver.
 
     .EXAMPLE
     Add-IBResourceRecordCName -IPv4Address 1.2.3.4 -HostName myserver.mydomain.com -GridServer myinfoblox.mydomain.com -Credential $Credential
 
     .PARAMETER IPv4Address
     The IPv4 address for the host record. Allows pipeline input.
 
     .PARAMETER HostName
     The hostname for the host record. Allows pipeline input.
 
     .PARAMETER GridServer
     The name of the infoblox appliance. Allows pipeline input.
 
     .PARAMETER Credential
     Add a Powershell credential object (created with for example Get-Credential). Allows pipeline input.
 
     #>
 
     [CmdletBinding()]
     param(
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [Alias('name')]
     $HostName,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     $Canonical,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [string] $GridServer,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [System.Management.Automation.PSCredential] $Credential)
 
     BEGIN { }
 
     PROCESS {
 
         $InfobloxURI = "https://$GridServer/wapi/v1.2.1/record:cname"
 
         $Data = "{`"name`":'$HostName',`"canonical`":'$Canonical'}" | ConvertFrom-Json | ConvertTo-Json
 
         $WebReqeust = Invoke-WebRequest -Uri $InfobloxURI -Method Post -Body $Data -ContentType "application/json" -Credential $Credential
 
         if ($WebReqeust.StatusCode -ne 201) {
             Write-Error "Request to Infoblox failed for record $HostName!"
             return
         }
     }
 
     END { }
 }
 
 function Add-IBResourceRecordA
 {
 
     <#
     .SYNOPSIS
     Adds a A record on the Infoblox Gridserver
 
     .DESCRIPTION
     This cmdlet creates a CName record on the Infoblox Gridserver.
 
     .EXAMPLE
     Add-IBResourceRecordCName -IPv4Address 1.2.3.4 -HostName myserver.mydomain.com -GridServer myinfoblox.mydomain.com -Credential $Credential
 
     .PARAMETER IPv4Address
     The IPv4 address for the host record. Allows pipeline input.
 
     .PARAMETER HostName
     The hostname for the host record. Allows pipeline input.
 
     .PARAMETER GridServer
     The name of the infoblox appliance. Allows pipeline input.
 
     .PARAMETER Credential
     Add a Powershell credential object (created with for example Get-Credential). Allows pipeline input.
 
     #>
 
     [CmdletBinding()]
     param(
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     $IPv4Address,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [Alias('name')]
     $HostName,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [string] $GridServer,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [System.Management.Automation.PSCredential] $Credential)
 
     BEGIN { }
 
     PROCESS {
 
         $InfobloxURI = "https://$GridServer/wapi/v1.2.1/record:a"
 
         $Data = "{`"ipv4addr`":'$IPv4Address',`"name`":'$HostName'}" | ConvertFrom-Json | ConvertTo-Json
 
         $WebReqeust = Invoke-WebRequest -Uri $InfobloxURI -Method Post -Body $Data -ContentType "application/json" -Credential $Credential
 
         if ($WebReqeust.StatusCode -ne 201) {
             Write-Error "Request to Infoblox failed for record $HostName!"
             return
         }
     }
 
     END { }
 }
 
 function Remove-IBResourceRecord
 {
 
     <#
     .SYNOPSIS
     Removes a host record from the Infoblox Gridserver
 
     .DESCRIPTION
     This cmdlet removes a host object from the Infoblox Gridserver.
 
     .EXAMPLE
     Get-InfoBloxRecord -RecordType host -RecordName MyHost -GridServer myinfoblox.mydomain.com -Credential $Credential -Passthrough | Remove-InfoBloxRecord
 
     .PARAMETER Reference
     The object reference for the host record. Allows pipeline input.
 
     .PARAMETER GridServer
     The name of the infoblox appliance. Allows pipeline input.
 
     .PARAMETER Credential
     Add a Powershell credential object (created with for example Get-Credential). Allows pipeline input.
 
     #>
 
     [CmdletBinding()]
     param(
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [Alias('_ref')]
     [string] $Reference,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [string] $GridServer,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [System.Management.Automation.PSCredential] $Credential)
 
     BEGIN { }
 
     PROCESS {
         $InfobloxURI = "https://$GridServer/wapi/v1.2.1/$Reference"
 
         $WebReqeust = Invoke-WebRequest -Uri $InfobloxURI -Method Delete -Credential $Credential
 
         if ($WebReqeust.StatusCode -eq 200) {
             $RecordObj = $WebReqeust.Content | ConvertFrom-Json
         }
         else {
             Write-Error "Request to Infoblox failed!"
             return
         }
     }
 
     END { }
 
 }
 
 
 function Set-IBResourceRecord
 {
 
     <#
     .SYNOPSIS
     Changes a host record on the Infoblox Gridserver
 
     .DESCRIPTION
     This cmdlet changes a host object on the Infoblox Gridserver.
 
     .EXAMPLE
     Set-InfoBloxRecord -Reference Reference -IPv4Address 1.2.3.4 -HostName myhost.mydomain.com -GridServer myinfoblox.mydomain.com -Credential $Credential
 
     .EXAMPLE
     Get-InfoBloxRecord -RecordType host -RecordName MyHost -GridServer myinfoblox.mydomain.com -Credential $Credential -Passthrough | Set-InfoBloxRecord -IPv4Address 2.3.4.5
 
     .PARAMETER IPv4Address
     The new IPv4 address for the host record.
 
     .PARAMETER HostName
     The new HostName for the host record.
 
     .PARAMETER Reference
     The object reference for the host record. Allows pipeline input.
 
     .PARAMETER GridServer
     The name of the infoblox appliance. Allows pipeline input.
 
     .PARAMETER Credential
     Add a Powershell credential object (created with for example Get-Credential). Allows pipeline input.
 
     #>
 
     [CmdletBinding()]
     param(
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [Alias('_ref')]
     [string] $Reference,
     [Parameter(Mandatory=$True)]
     [string] $IPv4Address,
     [Parameter(Mandatory=$False)]
     [string] $HostName,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [string] $GridServer,
     [Parameter(Mandatory=$True, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [System.Management.Automation.PSCredential] $Credential)
 
     BEGIN { }
 
     PROCESS {
         $InfobloxURI = "https://$GridServer/wapi/v1.2.1/$Reference"
 
         $Data = "{`"ipv4addrs`":[{`"ipv4addr`":'$IPv4Address'}] }" | ConvertFrom-Json | ConvertTo-Json
 
         $WebReqeust = Invoke-WebRequest -Uri $InfobloxURI -Method Put -Body $Data -Credential $Credential
 
         if ($WebReqeust.StatusCode -eq 200) {
             $RecordObj = $WebReqeust.Content | ConvertFrom-Json
         }
         else {
             Write-Error "Request to Infoblox failed!"
             return
         }
     }
 
     END { }
 
 }
 
 
 Export-ModuleMember *
`

