---
Author: atamido
Publisher: 
Copyright: 
Email: 
Version: 2.2
Encoding: ascii
License: cc0
PoshCode ID: 5877
Published Date: 2016-05-28t19
Archived Date: 2016-05-17t08
---

# powershell free/busy - 

## Description

query exchange with powershell for free/busy data

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 Import-Module -Name "C:\Program Files\Microsoft\Exchange\Web Services\2.2\Microsoft.Exchange.WebServices.dll"
 
   $Service = New-Object Microsoft.Exchange.WebServices.Data.ExchangeService
 $Service.UseDefaultCredentials = $true
 
 $Service.AutodiscoverUrl("john@example.com")
 
 #$Service.TraceEnabled = $true
 
 $StartTime = [DateTime]::Parse([DateTime]::Now.ToString("yyyy-MM-dd 0:00"))  
 $EndTime = $StartTime.AddDays(7)  
 
 $drDuration = new-object Microsoft.Exchange.WebServices.Data.TimeWindow($StartTime,$EndTime)  
 $AvailabilityOptions = new-object Microsoft.Exchange.WebServices.Data.AvailabilityOptions  
 $AvailabilityOptions.RequestedFreeBusyView = [Microsoft.Exchange.WebServices.Data.FreeBusyViewType]::DetailedMerged  
 $Attendeesbatch = New-Object "System.Collections.Generic.List[Microsoft.Exchange.WebServices.Data.AttendeeInfo]" 
 $attendee = New-Object Microsoft.Exchange.WebServices.Data.AttendeeInfo($userSMTPAddress) 
 
 #$Attendeesbatch.Add("dave@example.com")
 #$Attendeesbatch.Add("mike@contoso.com")
 
 $availresponse = ""
 $availresponse = $service.GetUserAvailability($Attendeesbatch,$drDuration,[Microsoft.Exchange.WebServices.Data.AvailabilityData]::FreeBusy,$AvailabilityOptions)
 
 $availresponse.AttendeesAvailability
 
 foreach($avail in $availresponse.AttendeesAvailability){
     foreach($cvtEnt in $avail.CalendarEvents){
         "Start : " + $cvtEnt.StartTime
         "End : " + $cvtEnt.EndTime
         "Subject : " + $cvtEnt.Details.Subject
         "Location : " + $cvtEnt.Details.Location
         ""
     }
 }
`

