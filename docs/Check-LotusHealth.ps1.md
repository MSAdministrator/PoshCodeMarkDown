---
Author: jeremy d. pavleck , pavleck.net
Publisher: 
Copyright: 
Email: 
Version: 4.1
Encoding: ascii
License: cc0
PoshCode ID: 723
Published Date: 
Archived Date: 2008-12-18t09
---

# check-lotushealth - check-lotushealth.ps1

## Description

check-lotushealth is a multi server, multi port ping script. originally designed to handle the port checks for a client’s entire lotus notes environment, i’ve removed the actual servers and replaced them with placeholders. you’ll need to adjust this to your environment.

## Comments



## Usage



## TODO



## script

`ping-port`

## Code

`#
 #
 
 Set-Variable -name SMTPPort -value 25 -option constant
 Set-Variable -name WWWPort -value 80 -option constant
 Set-Variable -name LDAPPort -value 389 -option constant
 Set-Variable -name Port410 -value 410 -option constant
 Set-Variable -name SSLPort -value 443 -option constant
 Set-Variable -name NOTESPort -value 1352 -option constant 
 
 $afails = @{}
 $astfails = @{}
 $agood = @{}
 
 $aservers = @{}
 $aservers["SERVER01"] = @{Notes=$True;}
 $aservers["SERVER02"] = @{SMTP=$True; WWW=$True; Notes=$True;}
 $aservers["SERVER03"] = @{SMTP=$True; WWW=$True; Notes=$True;}
 $aservers["SERVER04"] = @{WWW=$True; Notes=$True;}
 $aservers["SERVER05"] = @{WWW=$True; Notes=$True;}
 $aservers["SERVER06"] = @{Notes=$True;}
 $aservers["SERVER07"] = @{Notes=$True;}
 $aservers["SERVER08"] = @{Notes=$True;}
 $aservers["SERVER09"] = @{Notes=$True;}
 $aservers["SERVER10"] = @{Notes=$True;}
 $aservers["SERVER11"] = @{Notes=$True;}
 $aservers["SERVER12"] = @{Notes=$True;}
 $aservers["SERVER13"] = @{Notes=$True;}
 $aservers["SERVER14"] = @{Notes=$True;}
 $aservers["SERVER15"] = @{Notes=$True;}
 $aservers["SERVER16"] = @{Notes=$True;}
 $aservers["SERVER17"] = @{Notes=$True;}
 $aservers["SERVER18"] = @{Notes=$True;}
 $aservers["SERVER19"] = @{Notes=$True;}
 $aservers["SERVER20"] = @{SMTP=$True; Notes=$True;}
 $aservers["SERVER21"] = @{SMTP=$True; Notes=$True;}
 $aservers["SERVER22"] = @{SMTP=$True; Notes=$True;}
 $aservers["SERVER23"] = @{SMTP=$True; Notes=$True;}
 $aservers["SERVER24"] = @{WWW=$True; SSL=$True; Notes=$True;}
 $aservers["SERVER25"] = @{Notes=$True;}
 $aservers["SERVER26"] = @{WWW=$True; Notes=$True; SameTime=$True;}
 
 $astime = 1516, 1533, 8081, 8082, 1503, 554
 
 function Ping-Port([string]$server, [int]$port)
 {
 $tcpClient = New-Object System.Net.Sockets.TcpClient
 	$False
 	continue;
 }
 $tcpClient.Connect($server,$port)
 if ($tcpClient.Connected) {$True}           
 }
 
 function Ping-Server([string]$server) 
 {
 $ping = New-Object System.Net.NetworkInformation.Ping
    $False
     continue;
   }
 If ($ping.send($server).status -eq "Success") {$True}
 $ping = $null
 }
 
 $aservers.Keys | ForEach-Object {
  If (Ping-Server $_) {
  			Write-Host "$_ responded." -ForeGroundColor Green
  			$agood[$_] += @{HOST=$True;}
  					Write-Host "$_: Lotus Notes Port ($NOTESPort) Responding" -ForeGroundColor Green
  					$agood[$_] += @{NOTES=$True;}
  					} else {
  					Write-Host "$_: Host is reported to have Notes ($NOTESPort) but it is not responding." -ForeGroundColor Red
  					$afails[$_] += @{NOTES=$True;} 				
  					}
  				}
  					Write-Host "$_: SMTP ($SMTPPort) Responding" -Foregroundcolor green	
  					$agood[$_] += @{SMTP=$True;}
  					} else {
  					Write-Host "$_: Host is reported to have SMTP, but it is not responding." -Foregroundcolor red
  					$afails[$_] += @{SMTP=$True;}
  					}
  				}
  					Write-Host "$_: WWW ($WWWPort) Responding" -foregroundcolor green
  					$agood[$_] += @{WWW=$True;}
  					} else {
  					Write-Host "$_: Host is reported to have WWW, but it is not responding." -foregroundcolor red
  					$afails[$_] += @{WWW=$True;}
  					}
  				}
  					Write-Host "$_: LDAP ($LDAPPOrt) Responding" -foregroundcolor green
  					$agood[$_] += @{LDAP=$True;}
  					} else {
  					Write-Host "$_: Host is reported to have LDAP, but it is not responding." -foregroundcolor red
  					$afails[$_] += @{LDAP=$True;}
  					}
  				}
  					Write-Host "$_: Port 410 ($port410) Responding" -foregroundcolor green
  					$agood[$_] += @{Port410=$True;}
  					} else {
  					Write-Host "$_: Host is reported to have port 410, but it is not responding." -foregroundcolor red
  					$afails[$_] += @{Port410=$True;}
  					}
  				} 		
  					Write-Host "$_: SSL ($SSLPort) Responding" -foregroundcolor green
  					$agood[$_] += @{SSL=$True;}
  					} else {
  					Write-Host "$_: Host is reported to have SSL, but it is not responding." -foregroundcolor red
  					$afails[$_] += @{SSL=$True;}
  					}
  				}
  			  $st = $_
  			  $astime | ForEach-Object { 
  			  	If (Ping-Port $st $_) {
  			  		Write-Host "$st: SameTime Port $_ Success." -foregroundcolor green
  			  		} else {
  			  		Write-Host "$st: SameTime Port $_ Not Responding." -foregroundcolor red
  			  		$astfails[$st] += @{$_=$True;}
  			  		}
  			  	}
  			  }
  			} else {
  			Write-Host "Host $_ is not responding." -ForeGroundColor Red
  			$afails[$_] += @{HOST=$True;}
  			}
  }
 	
 
 If ($afails.count -gt 0) {
 	Write-Host "`n`nCompleted - Errors reported - the following ping tests failed:" -ForeGroundColor Magenta
 	Write-Host "`nServer `t`tFailed Ports" -Foregroundcolor Magenta
 	Write-Host "----------------------------------------" -ForegroundColor Magenta
 	$afails.Keys | ForEach-Object {
 			Write-host "$($_): `t$($afails[$_].Keys)" -ForeGroundColor Magenta
 			}
     } else {
     Write-Host "`n`nCompleted - All ports are responding." -ForegroundColor Green
 }
 
 If ($astfails.count -gt 0) {
 	Write-Host "Errors reported within the SameTime environment" -Foregroundcolor Magents
 	} else {
 	Write-Host "SameTime Environment is fully responsive." -ForegroundColor Green 
 }
`

