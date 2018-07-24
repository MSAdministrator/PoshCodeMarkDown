---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 698
Published Date: 
Archived Date: 2009-01-06t13
---

# get-activesyncusers - 

## Description

script to retreive all users with an active sync device partnership

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 $Mailboxes = Get-CASMailbox -Filter{HasActiveSyncDevicePartnership -eq $true}| select name, servername, DistinguisheDName, ActiveSyncMailboxPolicy
 
 $NumberWiped = 0
 $NumberSent =0
 $TotalCount = 0
 Foreach ($mailbox in $Mailboxes)
 {
 $Name = $mailbox.Name
 $DName= $mailbox.distinguisheDName
 $Stats = Get-ActiveSyncDeviceStatistics -mailbox $DName
 if ($Stats -ne $null)
 {	
 $bodySummary = $bodySummary + "<br />User "+ $mailbox.Name +" is on " +$mailbox.ServerName + " using the " +$mailbox.ActiveSyncMailboxPolicy + " policy"
 $TotalCount ++
 }
 }
 $body = "<font color=blue>Executive Summary:<br /><br />There are " + $TotalCount +" mailboxes that have a current Active Sync partnership:<br />"
 $body =	$body + $bodySummary
 $bodydetail = $bodydetail + "<br /><br />Device Information:" 
 Foreach ($mailbox in $Mailboxes)
 {
 $Name = $mailbox.Name
 $DName= $mailbox.distinguisheDName
 $Stats = Get-ActiveSyncDeviceStatistics -mailbox $DName
 if ($Stats.count -gt 1)
 {	
 $bodydetail = $bodydetail +"<br /><br />User "+ $mailbox.Name + " has the following " + $Stats.Count + " devices:<br />"
 }
 elseif($Stats.count -lt 2 -and $Stats -ne $null)
 {
 $bodydetail = $bodydetail +"<br /><br />User "+ $mailbox.Name + " has the following device:<br />"
 $bodydetail = $bodydetail + $Stats.count
 }
 if ($Stats.count -lt 2 -and $Stats -ne $null)
 {	
 if ($Stats.DeviceWipeRequestTime -ne $null)
 {
 if ($Stats.DeviceWipeAckTime -ne $null)
 {
 $NumberWiped ++
 $bodydetail = $bodydetail +"<br /><br />This device was wiped on " + $Stats.DeviceWipeAckTime.ToString()
 $bodydetail = $bodydetail +"<br />Click here to remove this active sync partnership:<a href= https://webmail.mt.gov/ExchActiveSync/RemoveActiveSyncConfirm.aspx?strIdentity=" + $Stats.Identity +"Remove Partnership>"
 }
 else
 {
 $NumberSent ++
 $bodydetail = $bodydetail +"><br /><font color=red><strong>This device was sent the wipe command on " + $Stats.DeviceWipeRequestTime.ToString() +"</strong></font color>"
 $bodydetail = $bodydetail +"<br />Click here to remove this active sync partnership:<a href= https://webmail.mt.gov/ExchActiveSync/RemoveActiveSyncConfirm.aspx?strIdentity=" + $Stats.Identity +">Remove Partnership"
 }
 }
 $bodydetail = $bodydetail + "<table><TR><TD>FirstSyncTime  </TD><TD>: " + $Stats.FirstSyncTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>LastPolicyUpdateTime  </TD><TD>: " + $Stats.LastPolicyUpdateTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>LastSyncAttemptTime   </TD><TD>: " + $Stats.LastSyncAttemptTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>LastSuccessSync       </TD><TD>: " + $Stats.LastSuccessSync +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceType            </TD><TD>: " + $Stats.DeviceType +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceID              </TD><TD>: " + $Stats.DeviceID +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceUserAgent       </TD><TD>: " + $Stats.DeviceUserAgent +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceWipeSentTime    </TD><TD>: " + $Stats.DeviceWipeSentTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceWipeRequestTime </TD><TD>: " + $Stats.DeviceWipeRequestTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceWipeAckTime     </TD><TD>: " + $Stats.DeviceWipeAckTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceModel           </TD><TD>: " + $Stats.DeviceModel +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceFriendlyName    </TD><TD>: " + $Stats.DeviceFriendlyName +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceOS              </TD><TD>: " + $Stats.DeviceOS +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceOSLanguage      </TD><TD>: " + $Stats.DeviceOSLanguage +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DevicePhoneNumber     </TD><TD>: " + $Stats.DevicePhoneNumber +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>Identity              </TD><TD>: " + $Stats.Identity +"</TD></TR></table><BR />"
 }
 elseif ($Stats.Count -gt 1)
 {
 foreach ($Stat in $Stats)
 {
 if ($Stat.DeviceWipeRequestTime -ne $null)
 {
 if ($Stat.DeviceWipeAckTime -ne $null)
 {
 $Wiped= ""
 $NumberWiped ++
 $bodydetail = $bodydetail +"<br /><br />This device was wiped on " + $Stat.DeviceWipeAckTime.ToString()
 }
 else
 {
 $NumberSent ++
 $bodydetail = $bodydetail +"<br /><STRONG><font color=red>This device was sent the wipe command on " + $Stat.DeviceWipeRequestTime.ToString()+"</STRONG></font color>"
 $bodydetail = $bodydetail +"<br />Click here to remove this active sync partnership:<a href= https://webmail.mt.gov/ExchActiveSync/RemoveActiveSyncConfirm.aspx?strIdentity=" + $Stats.Identity +">Remove Partnership"
 }
 }
 $bodydetail = $bodydetail + "<table><TR><TD>FirstSyncTime  </TD><TD>: " + $stat.FirstSyncTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>LastPolicyUpdateTime  </TD><TD>: " + $stat.LastPolicyUpdateTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>LastSyncAttemptTime   </TD><TD>: " + $stat.LastSyncAttemptTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>LastSuccessSync       </TD><TD>: " + $stat.LastSuccessSync +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceType            </TD><TD>: " + $stat.DeviceType +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceID              </TD><TD>: " + $stat.DeviceID +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceUserAgent       </TD><TD>: " + $stat.DeviceUserAgent +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceWipeSentTime    </TD><TD>: " + $stat.DeviceWipeSentTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceWipeRequestTime </TD><TD>: " + $stat.DeviceWipeRequestTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceWipeAckTime     </TD><TD>: " + $stat.DeviceWipeAckTime +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>LastPingHeartbeat     </TD><TD>: " + $stat.LastPingHeartbeat +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceModel           </TD><TD>: " + $stat.DeviceModel +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceIMEI            </TD><TD>: " + $stat.DeviceIMEI +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceFriendlyName    </TD><TD>: " + $stat.DeviceFriendlyName +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceOS              </TD><TD>: " + $stat.DeviceOS +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DeviceOSLanguage      </TD><TD>: " + $stat.DeviceOSLanguage +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>DevicePhoneNumber     </TD><TD>: " + $stat.DevicePhoneNumber +"</TD></TR>"
 $bodydetail = $bodydetail + "<TR><TD>Identity              </TD><TD>: " + $stat.Identity +"</TD></TR></table><BR />"
 }
 }
 }
 function sendmail([string] $body)
 {
 $SmtpClient = new-object system.net.mail.smtpClient 
 $MailMessage = New-Object system.net.mail.mailmessage 
 $SmtpClient.Host = "smtp.host" 
 $mailmessage.from = "From.Address@from.com" 
 $mailmessage.To.add("to.address@to.com") 
 $mailmessage.Subject = "Mailboxes that have an Active Sync Device Partnership" 
 $MailMessage.IsBodyHtml = $False
 $mailmessage.Body = $body
 $MailMessage.IsBodyHtml = $TRUE
 $smtpclient.Send($mailmessage) 
 }
 $body = $body + "<br /><br />There are $NumberWiped devices that have been wiped."
 $body = $body + "<br />There are $NumberSent devices that have a pending wipe command.</font color>"
 $Body = $Body + $BodyDetail
 sendmail($Body)
 $bodydetail = ""
 $Name = ""
 $bodySummary = ""
`

