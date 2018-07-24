---
Author: steve whitcher
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5907
Published Date: 2015-06-24t21
Archived Date: 2015-06-27t00
---

# renew certificates - 

## Description

fair warning

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 workflow Get-onlinecomputers
 {
     [CmdletBinding()]
     [Alias()]
     Param
     (
         [Parameter(Mandatory=$true,
                    ValueFromPipelineByPropertyName=$true,
                    Position=0)]
                    [Alias("ComputerName","cn")]
         $Name
     )
 
 
     foreach -parallel ($computer in $Name)
     {
         if (Test-connection -computername $computer -count 1 -erroraction SilentlyContinue) {
             $computer
         }
     
     }
 }
 
 $OnlinePCs = get-onlinecomputers $workstations.dnshostname
 
 $RenewCertificates = {
     Param([Datetime]$NewCACertDate)
     $Certs = get-childitem -path Cert:\LocalMachine\My
     $OldCerts = $Certs | Where-Object {$_.NotBefore -lt $NewCACertDate}
     foreach ($cert in $OldCerts)
     {
         $Serial = $cert.SerialNumber
         Write-Output $Serial
         certreq -enroll -machine -cert $Serial -q Renew ReuseKeys
     }
 }
 
 foreach ($computer in $OnlinePCs) {
         if ($computer) {
                     write-output $computer
                     invoke-command -computername $computer -ScriptBlock $RenewCertificates -argumentlist $NewCACertDate -authentication credssp -cred $cred
         }
     }
`

