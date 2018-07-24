---
Author: sankeerth ankala
Publisher: 
Copyright: 
Email: 
Version: 1.6
Encoding: ascii
License: cc0
PoshCode ID: 3831
Published Date: 2012-12-19t09
Archived Date: 2012-12-22t08
---

# process count checkalert - 

## Description

powershell script to verify process count check & email alert

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#*****************************************************************************************************************
   *  PowerShell Script to verify Process Count Check & Email alert
   *****************************************************************************************************************
     
     Script for checking a Process Life Cycle using Windows PowerShell
     Step 1) This Script checks if the Processes are Running
     Step 2) If the Total Process Count is less than 10 (that's my threshold - 10) - It would send you an Email with link to Auto Fix.
     Step 3) If the Total Process Count is greater or equal to 10- It would send a OK Email - no Action required.
     Step 4) I also included $Process2 and it being live and embedded the OR operator to check if either of them fail to send a Email Alert
     Step 5) If you have qns please email info@sankeerth.net
     
     Variables: ABC = $Process name and XYZ = $Process2 name , SMTP and From and TO Addresses 
                      and create Report.html & Fail Report.html to Email the to address.
   ******************************************************************************************************************
   *                 Version 1.6                   
   ******************************************************************************************************************
   *
   *      Last Modified :    Modtime: 12/18/2012 11:25a 
   *           Revision :    1.6
   *                 By :    Author: Sankeerth Ankala
   ******************************************************************************************************************#>
 
 $Process = GPS "ABC"
 $Process
 $Process.Count
 
 $Process2 = GPS "XYZ"
 $Process2.Count
 
 if(($Process.Count -lt 10) -or ($Process2.Count -eq 0))
 {
 $smtpServer = "SMTP" 
 $MailFrom = "fromemail@company.com"
 $mailto = "toemail@company.com"
 $msg = new-object Net.Mail.MailMessage  
 $smtp = new-object Net.Mail.SmtpClient($smtpServer)  
 $msg.From = $MailFrom 
 $msg.IsBodyHTML = $true 
 $msg.To.Add($Mailto)  
 $msg.Subject = "iBills Status Check - Fail" 
 $MailTextT =  Get-Content  -Path C:\temp\FailReport.html 
 $msg.Body = $MailTextT 
 $smtp.Send($msg) 
 }
 
 Else
 {
 $smtpServer = "SMTP" 
 $MailFrom = "fromemail@company.com"
 $mailto = "toemail@company.com"
 $msg = new-object Net.Mail.MailMessage  
 $smtp = new-object Net.Mail.SmtpClient($smtpServer)  
 $msg.From = $MailFrom 
 $msg.IsBodyHTML = $true 
 $msg.To.Add($Mailto)  
 $msg.Subject = "iBills Status Check - Success" 
 $MailTextT =  Get-Content  -Path C:\temp\Report.html 
 $msg.Body = $MailTextT 
 $smtp.Send($msg) 
 }
`

