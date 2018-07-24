---
Author: chriskenis
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3580
Published Date: 2012-08-14t04
Archived Date: 2012-08-17t15
---

# xc_smtpfunctions - 

## Description

report on all email addresses per mailbox. optionally remove regex matched addresses. add any custom address thru parsing any prefix in the loop of smtp domains.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 param(
 [Parameter(Position=0,ValueFromPipeline=$True)]
 [ValidateNotNullorEmpty()][string[]]$Mailboxes = @(),
 [string] $Regex = "domain\.local$|^CCMAIL|^MS:COMPREGION",
 $SMTPdomains = @("domain.com","sub.domain.com"),
 $Delimiter = "|",
 $OutputLog = "C:\Projects\XC\smtpadresses.csv",
 [switch] $Output,
 [switch]$Remove,
 [switch]$Update
 )
 
 Function NewSMTPAddress{
 param(
 $Mailbox,
 $Prefix,
 $Domain
 )
 $Address = "$Prefix@$Domain"
 $Mailbox | Set-Mailbox -EmailAddresses @{add = $Address}
 OutputMB -Mailbox $Mailbox -Address $Address -Match $False -Action "Added"
 }
 
 Function RemoveSMTPAddress{
 param(
 $Mailbox,
 $Address,
 $Match
 )
 $Mailbox | Set-Mailbox -EmailAddresses @{remove = $Address}
 OutputMB -Mailbox $Mailbox -Address $Address -Match $Match -Action "Removed"
 }
 
 Function OutputMB{
 param(
 $Mailbox,
 $Address,
 $Match,
 $Action
 )
 $global:objOut += New-Object -Typename PSObject -Property @{
 	Mailbox = $Mailbox
 	Address = $Address
 	Match = $Match
 	Action = $Action
 	}
 }
 
 $global:objOut = @()
 
 ForEach ($Mailbox in $Mailboxes){
 	try{
 		$sMailbox = get-mailbox -identity $Mailbox -EA Stop
 		foreach ($Address in $sMailbox.EmailAddresses){
 			if([regex]::IsMatch($Address,$Regex)){
 				if ($Remove){RemoveSMTPAddress -Mailbox $sMailbox -Address $Address -Match $True}
 				else{OutputMB -Mailbox $sMailbox -Address $Address -Match $True -Action "Matched"}
 				}
 			else{OutputMB -Mailbox $sMailbox -Address $Address -Match $False -Action "None"}
 		}
 		if ($Update){
 			foreach ($sDomain in $SMTPdomains){
 				NewSMTPAddress -Mailbox $sMailbox -Prefix $sMailbox.SamAccountName -Domain $sDomain
 				}
 			}
 		}
 	catch{write-warning "no such mailbox: $Mailbox";continue}
 }
 
 if($Output){
 	$objOut | Select Mailbox, Address, Match, Action | ConvertTo-Csv -Delimiter $Delimiter -NoTypeInformation | out-file $OutputLog -append
 	}
 else {$objOut | Select Mailbox, Address, Match, Action}
`

