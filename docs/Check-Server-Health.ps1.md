---
Author: brian english
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6748
Published Date: 2017-02-23t13
Archived Date: 2017-05-30t04
---

# check server health - 

## Description

ping-computer is not a standard powershell command…  replace line 54 with if (test-connection $srv -count 1).protocoladdress

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 ########################################################
 ########################################################
 ########################################################
 ########################################################
 param($Server)
 #################
 #################
 $GLOBAL:priority="normal"
 $FreePercent = 5
 $date = date
 #################
 
 #$SavedEA=$Global:ErrorActionPreference 
 #$Global:ErrorActionPreference="SilentlyContinue" 
 
 $inAry = @($input)
 if($inAry.count -gt 0)
 {$server = $inary}
 
 
 #############
 function checkSystem()
 { $status = "`nChecking Server Systems"    
   
   for($i = 0; $i -lt @($Server).count; $i++ ) 
   { $srv = @($Server)[$i]
     $checkVMWare = $false
     $checkDNS = $false
     $checkDHCP = $false
     $checkShares = $false
     $checkExc = $false
     $checkCitrix = $false
     $checkIIS = $false
     
     write-progress "Checking Server Systems: $srv "-> -perc ($i * (100/@($Server).count))
     
     $status += "`n***$srv***"
     $errcnt = 0
     $error.clear()
     
     @@if((test-connection "$srv" -count 1).protocoladdress -ne "")
     {
       if((gwmi -computername $srv win32_computersystem).dnshostname -ne $srv -or (gwmi -computername $srv win32_computersystem).name -ne $srv)
       { $status += "`n`t! ServerName does not match"
         $errcnt += 1
       }
     
       foreach($drive in(gwmi win32_logicaldisk -computername $srv | where{$_.Drivetype -like "3"}))
       { if((($drive.freespace/1gb)/($drive.size/1gb) * 100 ) -lt $FreePercent)
         { $status += "`n`t! Drive: " + $drive.deviceid + " Free Space: " + ($drive.freespace/ 1GB) + "GB"
           $priority = "high"
           $errcnt += 1
         }
       }
       
       foreach($svc in(gwmi win32_service -computername $srv | where{$_.StartMode -like "Auto" -and $_.name -ne "SysmonLog"}))
       { if($svc.state -ne "Running")
         { $status += "`n`t! Service: " + $svc.DisplayName + " is " + $svc.state
           $errcnt += 1 
         }
         if($svc.name -eq "vpxd" -and $svc.state -eq "Running")
         {$checkVMWare = $true}
         if($svc.name -eq "DNS" -and $svc.state -eq "Running")
         {$checkDNS = $true}
         if($svc.name -eq "DHCPServer" -and $svc.state -eq "Running")
         {$checkDHCP = $true}
         if($svc.name -eq "lanmanserver" -and $svc.state -eq "Running")
         {$checkShares = $true}
         if($svc.name -eq "MSExchangeIS" -and $svc.state -eq "Running")
         {$checkExc = $true}
         if($svc.name -eq "cpsvc" -and $svc.state -eq "Running")
         {$checkCitrix = $true}
         if($svc.name -eq "IISADMIN" -and $svc.state -eq "Running")
         {$checkIIS = $true}
       }
           
       if($checkVMWare)
       { $vi = get-esx $srv
         
         $status += "`nHosts"    
         foreach($vmh in get-vmhost)
         { if($vmh.state -ne "Connected")
           { $status += "`n`t! Host: " + $vmh.Name + "; State: " + $vmh.state  }
         }
         
         $status += "`nVMs"
         $vms = (get-vm -server $vi)      
         $greenvms = 0
         
         $status += "`n`tTotal VMs: " + $vms.count
         $status += "`n`tVMs powered Off: " + ($vms | where{$_.PowerState -eq "PoweredOff"}).count
         $status += "`n`tVMs powered On: " + ($vms | where{$_.PowerState -eq "PoweredOn"}).count
             
         foreach($vm in ($vms | where{$_.PowerState -eq "PoweredOn"}))
         { $vmv = get-view $vm.id
           
           { $status += "`n`t`t" + $vm.name + " Overall: " + $vmv.overallStatus
             $status += "`n`t`t" + $vm.name + " Config: " + $vmv.configStatus
             $status += "`n`t`t" + $vm.name + " Heartbeat: " + $vmv.GuestHeartbeatStatus
             
             foreach($alrm in $vmv.triggeredAlarmState)
             { $av = get-view $alrm.alarm
               $status += "`n`t`t " + $vm.name + " Triggered: " + $av.info.Name
             }
             $priority = "high"
           }
           else
           {$greenvms += 1}              
         $status += "`n`tGreen VMs: " + $greenvms + "`n"
       }
       
       if($checkDNS)
       { foreach($ln in (dcdiag /test:DNS /s:$srv /v))
         { if( $ln -like "*warning:*" -or $ln -like "*error:*") 
           {$status += "`n`t" + $ln.trim()}
         }
         foreach($ln in (dcdiag /test:CheckSecurityError /s:$srv /v))
         { if($ln -like "*warning:*" -or $ln -like "*error:*") 
           {$status += "`n`t" + $ln.trim()}
         }
       
       if($checkDHCP)
       {
         ##########################
       
       if($checkShares)
       { foreach($share in (gwmi win32_share -computername $srv | where {$_.status -ne "OK"}))
         { $status += "`n`t! Share: " + $share.Name + " is " + $share.status
           $errcnt += 1
         }      
       
       if($checkExc)
       { foreach($exc in (gwmi ExchangeConnectorState -namespace "root\cimv2\applications\exchange" -computername $srv | where {$_.IsUp -ne $true}))
         { $status += "`n`t! Exchange Connector: " + $exc.Name + " is DOWN"
           $errcnt += 1
         }
         
         foreach($exc in (gwmi ExchangeQueue -namespace "root\cimv2\applications\exchange" -computername $srv | where {$_.NumberOfMessages -gt 1}))
         { $status += "`n`t! Exchange Queue: " + $exc.QueueName + " has " + $exc.NumberofMessages
           $errcnt += 1
         }
         
         foreach($exc in (gwmi Exchange_Logon -namespace "root\microsoftexchangev2" -computername $srv | where {$_.Latency -gt 60000}))
         { $status += "`n`t! Exchange Logons: " + $exc.MailboxDisplayName + " in Store " + $exc.StoreName + " has a latency of " + $exc.Latency
           $errcnt += 1
         }
         
       
       if($checkCitrix)
       { $mfserver = New-Object -com "MetaframeCom.MetaFrameServer"
         $mfserver.initialize(6,$srv)
         $status += "`n`t Sesion Count " + $mfserver.SessionCount
         
         switch($mfserver.WinServerObject.EnableLogon)
         {
           "0" { $status += "`n`t! Not Allowing Connections"
                 $errcnt += 1
               }
           "1" {}
         }
       
       if($checkIIS)
       { $pathToTest = "\\$srv\c$\windows\system32\inetsrv\metabase.xml"
         if(test-path $pathToTest)
         { $mb=[xml](get-content $pathToTest)
         
           $WebSites = $mb.configuration.MBProperty.IIsWebServer 
           $WebVDirs = $mb.configuration.MBProperty.IIsWebVirtualDir
           
           If(!($mb.configuration.MBProperty.IIsWebService.Custom | where {$_.name -eq "ServerComment"}).value) 
           { $WebServerName = "[UNKNOWN]"  } 
           Else 
           { $WebServerName = ($mb.configuration.MBProperty.IIsWebService.Custom | where {$_.name -eq "ServerComment"}).value }
           
           If($WebServerName -ne $srv) 
           { $status += "`n`t! WebServer Name Mismatch $WebServerName, $srv"  
             $errcnt += 1
           }
         
           ForEach ($Site in $WebSites) 
           { if($Site.ServerComment -notlike "Allows*")
             { $sitePath = "$srv" + ($site.location -replace "/LM","")
               $siteState = ([ADSI]("IIS://$sitepath")).serverstate
               if($siteState -ne "2")
               { switch($sitestate)
                 { "1" {$siteState = "starting"}
                   "2" {$siteState = "started"}
                   "3" {$siteState = "stopping"}
                   "4" {$siteState = "stopped"}
                   "5" {$siteState = "pausing"}
                   "6" {$siteState = "paused"}
                   "7" {$siteState = "continuing"}
                 }
                 $status += "`n`t! " + $Site.ServerComment + " is not running: state " + $sitestate
                 $errcnt += 1
               }
             }
           }
         
         }   
         else
         { $status += "`n`t! Unable to verify IIS Metabase"
           $errcnt += 1
         }
     }
     else
     { $status += "`n`t! Server Unreachable"
       $errcnt += 1
     }
     
     if($error[0])
     { $status += "`n`t Errors Occured: "
       foreach($err in $error)
       { $status += "`n`t`!" + $err }
     } 
     elseif($errCnt -eq 0)
     {$status += "`n***$srv*** Healthy" }
     else
     {$status += "`n***$srv*** Unhealthy"}
     $status
     $status = ""
 #############
 
 ########################################################
 ############
 
 checkSystem 
 ########################################################
 #$Global:ErrorActionPreference=$SavedEA
`

