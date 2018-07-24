---
Author: stefan stranger
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2746
Published Date: 2011-06-19t13
Archived Date: 2011-08-09t07
---

# ds l order brneanew game - 

## Description

retrieve dell order status

## Comments



## Usage



## TODO



## script

``

## Code

`#
 
 $wc = New-Object System.Net.WebClient
 $url = 'http://support.euro.dell.com/support/order/emea/OrderStatus.aspx?c=nl&l=nl&s=gen&ir=NL0131-8510-29070&ea=stefan.stranger@getronics.com'
 $RawResult = $wc.DownloadString($url)
 
 $r = New-Object regex('Lever.*?(\d+-\d+-\d{4})</B></TD></TR>','SingleLine' )
 $m = $r.Matches($RawResult)
 $date = ($m |% {$_.groups[1].value })
 $r = New-Object regex('target="popup:640x480">(.*?)</a>'  )
 $m = $r.Matches($RawResult)
 $status = ($m |% {$_.groups[1].value }) 
 $Orderstatus = 1 | select @{name='Status';expression={$status[0]}},@{name='Estimated Delivery Date';expression={$date}}
 $Orderstatus | ft -a
 #[string]$output = $orderstatus
 $startdate = "16-09-2007"
 $startdate
 $currentdate = (get-date).ToString('MM-dd-yyyy')
 $currentdate
 $date
`

