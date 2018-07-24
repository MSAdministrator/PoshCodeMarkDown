---
Author: david woods
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 4990
Published Date: 2016-03-17t00
Archived Date: 2016-04-23t15
---

# user termination script - 

## Description

the script below has 3 different components & was designed for use by our service desk. when a user is terminated, the script will remove them from lync, remove all mailbox information (forwarding rules etc) in exchange 2010, disable ad object, remove group membership & move to “disabled users” ou.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 $blnCheckSnappin = Get-PSSnapin | where {$_.Name -eq "microsoft.exchange.management.powershell.e2010"}
 
 if ($blnCheckSnappin -eq $null)
 {    
     add-pssnapin "microsoft.exchange.management.powershell.e2010" -ErrorVariable errSnapin ;
     . $env:ExchangeInstallPath\bin\RemoteExchange-mod.ps1
     Connect-ExchangeServer -Server <YourExchangeServer> -allowclobber
 }
     
 Import-Module "$env:programfiles\Common Files\Microsoft Lync Server 2010\Modules\Lync\Lync.psd1"
 Import-Module ActiveDirectory
 
 function terminateUser 
 {   
 
     $strMsg1 = ""
     $strMsg2 = ""
     $strMsg3 = ""
     $strMsg4 = ""
     $strMsg5 = ""
     $strUserID = $args[0]
     
     try
     {
         $blnUserExists = get-aduser $strUserID
     }
     catch
     {
 
     }    
 
     if($blnUserExists)
     {
         $strMsg3 += "`r`n` "
     	$strMsg3 += "Are you sure you wish to continue? (Y/N) `r`n` "        
         $strMsg3 += "`r`n` "
 
         $strContinue = read-host $strMsg3
         
         if($strContinue -eq "Y")
         {   			
             $strRunLog = "$strRunDir\Logs\$strUserID-log-$strDate.txt"
             $arrAttachments = @($strRunLog)
             
             start-transcript -path $strRunLog | Out-null
 
                         
             Disable-CSuser -identity $strUserID 
              
 
             Get-InboxRule -Mailbox $strUserID  | Remove-InboxRule -Confirm:$false -Force        
             Set-CASMailbox -Identity $strUserID -ActiveSyncEnabled:$False -OWAEnabled:$False
             Get-ActiveSyncDevice �Mailbox $strUserID | Remove-ActiveSyncDevice -Confirm:$False          
             Set-mailbox -Identity $strUserID -HiddenFromAddressListsEnabled:$true -ForwardingAddress $Null
             
                     
             
             Get-ADPrincipalGroupMembership -Identity $strUserID | Select-object name | format-table
                                      
             Get-ADPrincipalGroupMembership -Identity $strUserID | Where {$_.Name -ne "Domain Users"} | ForEach-Object {Remove-ADPrincipalGroupMembership -Identity $strUserID -MemberOf $_.SamAccountName -Confirm:$False}
             
             Set-ADuser -identity $strUserID -Manager $Null
             Disable-ADAccount -identity $strUserID 
             Get-ADUser -identity $strUserID | Move-ADobject -TargetPath "OU=Disabled,OU=Users,OU=Client,DC=YOURCOMPANY,DC=com,DC=au"                                
             
             $strMsg4 += "`r`n` "    		
             $strMsg4 += "The Termination script for",$strUserID,"has completed. `r`n` "
             $strMsg4 += " `r`n` "
             $strMsg4 += "A log of the session can be located at: $strRunLog. `r`n` "
             $strMsg4 += "`r`n` "    		
             
             Write-host $strMsg4                                                     
             
             $strSubject = "Account Termination Log - $strUserID - Operation Successful"                                             
                         
             stop-transcript | Out-null
             
             start-process notepad.exe $strRunLog
                         
             send-MailMessage -SmtpServer $strSMTPserver -To $arrRecipient -From $strFrom -Subject $strSubject -attachments $arrAttachments -Priority high 
             
             $strMsg5 += "`r`n` "
         	$strMsg5 += "User account terminated. Do you wish to terminate another account? (Y/N) `r`n` "
             $strMsg5 += "`r`n` "
             $strMsg5 += "`r`n` "
 
             $strContinue = read-host $strMsg5
             
             if($strContinue -eq "Y")
             {
                 enterUser
             }
             else
             {
                 [System.Windows.Forms.MessageBox]::Show("Exiting Script. ","Status")    
             }                
         }
         else
         {
             [System.Windows.Forms.MessageBox]::Show("Exiting Script. ","Status")  
         }        
 
     }
     else
     {
         [System.Windows.Forms.MessageBox]::Show("ERROR. $strUserID does not exist in AD. Exiting script. ","Status")  
     }
  }    
  
  function enterUser
  {
  
      $strMsg2 += "`r`n` "
      $strMsg2 += "Please enter the user's AD Account Below: `r`n` "
      $strMsg2 += "`r`n` "
      
      $strUserID = read-host $strMsg2
      $strUserID = $strUserID.Trim()
 
      terminateUser $strUserID
  }        
 
 [console]::ForegroundColor = "yellow"
 
 #
 #
 #
 
 [System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") | Out-null
 
 #
 #
 
 $StrInvocation = (Get-Variable MyInvocation).Value
 $strRunDir = Split-Path $StrInvocation.MyCommand.Path
 $strDate = get-date -format "dd-MMM-yyyy-HHmm"
 $blnUserExists = $false
 $strUserID = ""
 $strSMTPserver = "<YourMailServer>"  
 $strFrom = "<YourAddress>"    
 $arrRecipient = @("<YourRecipients>")
 $strSubject = ""
 $strDebug = ""
 $strOperator = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name
 
 $strMsg1 += "`r`n` "
 $strMsg1 += "User Termination Process Script for <YourCompany> `r`n` "
 $strMsg1 += "`r`n` "
 $strMsg1 += "Created by EBS Platforms Team - 2014 `r`n` "
 $strMsg1 += "`r`n` "
 $strMsg1 += "This script will accomplish the following: `r`n` "
 $strMsg1 += "`r`n` "
 $strMsg1 += "1.) Disable Lync Account (Lync 2010) `r`n` "
 $strMsg1 += "2.) Remove Inbox Rules (Exchange 2010) `r`n` "
 $strMsg1 += "3.) Remove all Mail Forwarding Rules (Exchange 2010) `r`n` "
 $strMsg1 += "4.) Hide email address from Exchange Address Lists (Exchange 2010) `r`n` "
 $strMsg1 += "5.) Remove Group Membership (including DL) from AD object (Active Directory) `r`n` "
 $strMsg1 += "6.) Remove Manager Field from AD object (Active Directory) `r`n` "
 $strMsg1 += "7.) Disable Account (Active Directory) `r`n` "
 $strMsg1 += "8.) Move disabled AD object to Disabled OU `r`n` "
 
 write-host $strMsg1
 
 enterUser
`

