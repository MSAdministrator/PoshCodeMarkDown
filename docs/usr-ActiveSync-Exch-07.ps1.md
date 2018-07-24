---
Author: pedro genil
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: utf-8
License: cc0
PoshCode ID: 3785
Published Date: 2012-11-26t00
Archived Date: 2012-11-30t08
---

# usr activesync - exch 07 - 

## Description

sacamos el numero de dispositivos active sync en una organizacion exchange 2007.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 If ((Get-PSSnapin | where {$_.Name -match "Exchange.Management"}) -eq $null)
 {
 	Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Admin
 }
 $mailbox = Get-MailboxServer
 $fecha = get-date 
 $fecha= $fecha.adddays(-30).ToString("yyyyMMdd")
 foreach ($mail in $mailbox)
 {
 $a=0
 $b=0
 echo "Analizando $mail" >> resultado.txt
 $devices = Get-Mailbox -server $mail -resultsize unlimited| ForEach {Get-ActiveSyncDeviceStatistics -Mailbox:$_.Identity} 
 foreach ($device in $devices)
    {
      if($device.LastSuccessSync -ne $NULL)
           {
            if ($device.LastSuccessSync.ToString("yyyyMMdd") -gt $fecha)
             {$a=$a+1}
           }
            else
             {$b=$b+1}
           }
 echo "Numero de dispositivos nunca conectados en $mail $b" >> resultado.txt
 echo "Numero de dispositivos conectados en los ultimos 30 dias en $mail $a" >> resultado.txt           
 }
`

