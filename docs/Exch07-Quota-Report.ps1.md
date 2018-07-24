---
Author: hinkle
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2117
Published Date: 
Archived Date: 2010-09-05t00
---

# exch07 quota report - 

## Description

power shell 1 script used to grab mailbox stats for a exchange 2007 server.  it saves the information into a .csv file by changing the $outfile variable.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 
 $date = get-date -Format MM-dd-yyyy
 
 $OUTFILE = "C:\Net_Admin_Stuff\usage_reports\daily_storage_limits\Daily_Storage_Limits-$date.csv" 
 
 $DEFAULTSENDQUOTA = 510000/1024
 
 $HEADER = "Display Name,Storage Limit Status,Item Count,Total Item Size (MB),Deleted Item Count,Total Deleted Item Size (MB),Prohibit Send/Receive Quota (MB),Quota Source"
 $HEADER | Out-File $OUTFILE -Append -Encoding Ascii
 
 Get-MailboxStatistics |  where {$_.ObjectClass -eq 'Mailbox'} | Sort-Object StorageLimitStatus | ForEach-Object {		
 	$CURUSER = get-mailbox -Identity $_.Identity   
 
 	If ($CURUSER.UseDatabaseQuotaDefaults -eq $true) 
 	{
  		$QUOTASRC = $CURUSER.Database	  
 		$SENDQUOTA = $DEFAULTSENDQUOTA	
 	}   
 	else  
 	{    
 		$QUOTASRC = "User Profile"    
 		$SENDQUOTA = $CURUSER.ProhibitSendReceiveQuota.Value.ToMB()	  
 	}	
 
 	$DNAME = $_.DisplayName;	
 	$SLSTATUS = $_.StorageLimitStatus;	
 	$ICOUNT = $_.ItemCount;	
 	$TISIZE = $_.TotalItemSize.Value.ToMB();	
 	$DICOUNT = $_.DeletedItemCount;	
 	$TDISIZE = $_.TotalDeletedItemSize.Value.ToMB();	
 
 	"$DNAME,$SLSTATUS,$ICOUNT,$TISIZE,$DICOUNT,$TDISIZE,$SENDQUOTA,$QUOTASRC" 
 } | Out-File $OUTFILE -Append -Encoding Ascii
`

