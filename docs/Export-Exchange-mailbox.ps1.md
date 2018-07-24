---
Author: jason m
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5143
Published Date: 2015-05-05t05
Archived Date: 2015-12-06t19
---

# export exchange mailbox - 

## Description

this script will take members of an ad group and export their exchange mailbox to a pst as well as export all lync archives for the specified time period. the script can be ran manually or bet set to run as a scheduled task. requires the quest ad powershell module as well as lync, exchange, and ad modules to run.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###################################################################################################
 ###################################################################################################
  
 ###################################################################################################
 ###################################################################################################
  
 if ((get-pssnapin | % { $_.name }) -notcontains "Quest.ActiveRoles.ADManagement")
 { Add-PSSnapin Quest.ActiveRoles.ADManagement }
  
 if ((get-pssnapin | % { $_.name }) -notcontains "Microsoft.Exchange.Management.PowerShell.E2010")
 { Add-PSSnapin Microsoft.Exchange.Management.PowerShell.E2010 }
  
 if ((get-module | % { $_.name }) -notcontains "ActiveDirectory")
 { Import-Module activedirectory }
 if ((get-module | % { $_.name }) -notcontains "Lync")
 { Import-Module 'C:\Program Files\Common Files\Microsoft Lync Server 2013\Modules\Lync\Lync.psd1' }
 ###################################################################################################
 ###################################################################################################
  
 $holdgroup = "ExportGroup"
  
 $workfile = "workfile.csv"
  
 $logfile = "exportlog.txt"
  
 $MBUserPerm = "Domain\User"
  
 $BadItems = "10"
  
 $destpath = "\\server.domain.com\Exports"
  
 $locpst = "D:\PST"
  
 $dt = get-date -uformat "%m-%d-%Y-%H-%M"
  
 $date = Get-Date -format MMM-dd-yyyy
  
 $LyncDateStart = (get-date).addDays(-90).ToString("MM/dd/yyy")
  
 $LyncDateEnd = (get-date).ToString("MM/dd/yyy")
  
 $LyncSQL = "ArchivingDatabase:sqlserver.domain.com"
  
 get-qadgroupmember $holdgroup | get-qaduser | Select-Object FirstName, LastName, SamAccountName, HomeDirectory, PrimarySMTPAddress | Export-Csv -NoTypeInformation $workfile
  
  
 ######################################################################################################
 ######################################################################################################
 $copypath = Import-Csv $workfile
 ForEach ($User in $copypath)
 {
     $strA = $destpath
     $strB = $user.Firstname
     $strC = $user.LastName
     $strD = $strA + "\" + $strB + "_" + $strC
     $strE = $locpst
     $strF = $strE + "\" + $strB + "_" + $strC
     $pathcheck0 = Test-Path $strD
     $pathcheck1 = Test-Path $strF\mailbox\*
     $pathcheck2 = Test-Path $strF
     
     #######################################################################################################
     #######################################################################################################
     
     Export-CsArchivingData -Identity "$LyncSQL" -StartDate "$lyncdateStart" -EndDate $LyncDateEnd -UserUri $User.PrimarySMTPAddress -OutputFolder "$strD\Lync_Archive\$date"
     
     
     #######################################################################################################
     #######################################################################################################
     if ($pathcheck1 -eq $True)
     {
         robocopy $strF\mailbox\ $strD\mailbox /Copy:DAT /E /XO /ZB /MT /R:2 /W:2 /V /TS /FP /LOG+:$logfile
         remove-QADGroupMember $holdgroup $User.SamAccountName
         Remove-Item -Recurse -Force $strF
     }
     else
     {
         if ($pathcheck2 -eq $true)
         {
             Write-Host "PST export pending."
         }
         else
         {
             mkdir $strF\Mailbox
             Add-MailboxPermission $User.SamAccountName -accessrights fullaccess -user $MBUserPerm -confirm:$False
             New-MailboxExportRequest -Mailbox $User.SamAccountName -Name "ExportToPST-$dt" -FilePath "$strF\mailbox\Mailbox--$Date.pst" -badItemLimit $BadItems -confirm:$False
             Remove-MailboxPermission $User.SamAccountName -accessrights fullaccess -user $MBUserPerm -confirm:$False
         }
     }
     
     Start-Sleep 120
     
     #######################################################################################################
     #######################################################################################################
     
     del $workfile
     del $logfile
`

