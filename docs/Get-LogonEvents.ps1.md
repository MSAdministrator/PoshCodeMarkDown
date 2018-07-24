---
Author: david mcdonough
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6701
Published Date: 2017-01-18t20
Archived Date: 2017-03-31t03
---

# get-logonevents - 

## Description

retrieve user logon events (relatively) quickly. this will output results to the console as they are retrieved

## Comments



## Usage



## TODO



## function

`get-logonevents`

## Code

`#
 #
 function Get-LogonEvents{
 
 [CmdletBinding()]
 
     param (
 
         [Parameter(Mandatory=$true, Position=0, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
         [Alias('ServerName','Server','Name')]
         [string[]]$ComputerName,
 
         [Parameter(ValueFromPipelineByPropertyName=$true,Mandatory=$true)]
         [PSCredential]$Credential,
 
         [Parameter()]
         [ValidateSet("Service","Interactive","RemoteInteractive","NetworkCleartext","CachedInteractive","Unlock","NewCredentials","Network","*")] 
         [string[]]$LogonType = @("Interactive","RemoteInteractive","CachedInteractive"),
 
         [string]$UserName,
 
         [Parameter()]
         [switch]$Oldest,
         
         [Parameter()]
         [int64]$MaxEvents,
 
         [Parameter()]
         [datetime]$StartTime=(Get-Date 1/1/1900),
 
         [Parameter()]
         [datetime]$StopTime=(Get-Date 1/1/2100)   
         
     )
 
     Begin{
 
 
         Function ParseEventMessage
         {
             [CmdletBinding()]
             
             param( 
                 [Parameter(ValueFromPipeline=$true)]
                 $obj
             )
 
             Begin{
                 $defaultDisplaySet = 'MachineName','Result','TimeCreated','TargetUserName'
                 $defaultDisplayPropertySet = New-Object System.Management.Automation.PSPropertySet(�DefaultDisplayPropertySet�,[string[]]$defaultDisplaySet)
                 $PSStandardMembers = [System.Management.Automation.PSMemberInfo[]]@($defaultDisplayPropertySet)
                 $myHash = @{}
             }
             Process{
                 
                 $myHash['Result'] = switch ($obj.id)
                 {
                     4625 { "Failure" }
                     4624 { "Success" }
                 }
                 $myHash['ID'] = $obj.ID
                 $myHash['TimeCreated'] = $obj.TimeCreated
                 $myHash['MachineName'] = $obj.MachineName
 
 
                 New-Object -TypeName PSObject -Property $myHash | ForEach-Object { 
                     
                     $PSItem.PSObject.TypeNames.Insert(0,"EventLogRecord.XMLParse")
 
                     $PSItem | Add-Member MemberSet PSStandardMembers $PSStandardMembers -PassThru 
                 }
 
             }
         }
 
         $hashLogonType = @{
 
             Interactive = 2
             Network = 3
             Service = 5
             Unlock = 7
             NetworkCleartext = 8
             NewCredentials = 9
             RemoteInteractive = 10
             CachedInteractive = 11
 
         }
 
     $filter = @"
 <QueryList>
     <Query Id="0" Path="Security">
     <Select Path="Security">
         *[System[
             (EventID=4624 or EventID=4625)            
             and TimeCreated[@SystemTime&gt;='{0}' and @SystemTime&lt;='{1}']
         ] 
             and EventData[
                 Data[@Name='LogonType'] and ({2})
                 {3}
             ]
         ]
     </Select>
     </Query>
 </QueryList>
 "@
 
     }
   
     Process{
         
         foreach ($obj in $ComputerName)
         {            
             
             if ($UserName)
             {
                 $joinUserName = "and Data[@Name='TargetuserName'] and (Data='{0}')" -f $UserName
             }
 
             $joinLogonType = if ($LogonType -eq '*')
             {
             
                 $hashLogonType.Values -replace '^',"Data='" -replace '$',"'" -join " or "
             
             }
             Else
             {
                 $($LogonType | ForEach-Object { $hashLogonType[$PSItem] }) -replace '^',"Data='" -replace '$',"'" -join " or "
             }
 
             $objFilter = $filter -f (Get-Date $StartTime -Format s),(Get-Date $StopTime -Format s),$joinLogonType,$joinUserName
            
             $hashEventParm = @{ 
                 ComputerName = $obj
                 FilterXml = $objFilter
             }
 
             if ($Credential) { $hashEventParm['Credential'] = $Credential }
             if ($MaxEvents) { $hashEventParm['MaxEvents'] = $MaxEvents }
 
             $objFilter | Write-Verbose
 
             Get-WinEvent @hashEventParm | ParseEventMessage
 
         }
 
     }
 
 
     End{}
 
 }
`

