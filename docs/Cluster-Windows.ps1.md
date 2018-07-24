---
Author: pedro genil
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3787
Published Date: 2012-11-26t00
Archived Date: 2012-11-30t08
---

# cluster windows - 

## Description

comprobamos el estado de los cluster de windows.

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 #########################################
 If ((Get-PSSnapin | where {$_.Name -match "Exchange.Management"}) -eq $null)
 {
 	Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Admin
 }
 $activos= Get-content "F:\Scripts\Cluster\activos.txt"
 $notabla = @()
 $notabla += "<table width='200' border='2' cellspacing='0'>"
 $contador = 0
     foreach($activo in $activos)
     {
     $datos= gwmi -q "select * from MSCluster_ResourceGroup" -namespace root\mscluster -computername $activo -Authentication PacketPrivacy | select Name,InternalState,State,__SERVER 
     $name = gwmi -q "select * from MSCluster_ResourceGroup" -namespace root\mscluster -computername $activo -Authentication PacketPrivacy | where {$_.name -like "*CLUS*"} | Select name
     $namecluster= $name.name
     foreach ( $propdatos in $datos)
                         {
                           $ClusterStatus = Get-ClusteredMailboxServerStatus -Identity $namecluster | Select -Expand OperationalMachines | ForEach {If($_ -like "*Owner*") {$_}}
                           $ActiveNode = $ClusterStatus.Split(" ")[0]
                           if ($ActiveNode -eq $propdatos.__SERVER)
                               {
                               if ($propdatos.name -ne "Available Storage")
                               {
                                 if ($propdatos.internalstate -eq "Offline")
                                     {
                                         $notabla += "<tr><td>" + $propdatos.name + "</td><td><font color='red'>" +  $propdatos.internalstate + "</font></td><td>" + $activeNode +"</td></tr>"
                                         $contador = $contador + 1
                                     } 
                                 else
                                     {
                                     if ($contador -gt 0)
                                     {
                                         $notabla += "<tr><TH COLSPAN=3> <font color='blue'>" + $namecluster + "</th></font></tr>"
                                         
                                      }   
                                     }
                               }
                               } 
                            else
                                { 
                                
                                $contador = $contador + 1 
                                
                                if ($propdatos.name -ne "Available Storage")
                                {
                                if ($propdatos.internalstate -eq "Offline")
                                     {                                       
                                         
                                         $notabla += "<tr><td>" + $propdatos.name + "</td><td><font color='red'>" +  $propdatos.internalstate + "</font></td><td BGCOLOR='red'>" + $activeNode +"</td></tr>"
                                     }                      
                                 else
                                     {
                                         
                                         $notabla += "<tr><td>" + $propdatos.name + "</td><td><font color='red'>" +  $propdatos.internalstate + "</font></td><td BGCOLOR='red'>" + $activeNode +"</td></tr>"
                                     }
                                }
                               }
                                  
                               
                         }
 
 $notabla += "</table><br>" 
 $notabla > estado.html
 if ($contador -gt 0)
 {
     $smtpServer = "xx.xx.xx.xx"
 	$smtpFrom = "Cluster Windows <email@dominio.es>"
 	$smtpTo = "email2@dominio.es"
     $message = New-Object System.Net.Mail.MailMessage $smtpfrom, $smtpto
 	$message.Subject = "Chequeo Cluster Windows"
 	$message.IsBodyHTML = $true
     $message.Body = $notabla
 	$smtp = New-Object Net.Mail.SmtpClient($smtpServer)
 	$smtp.Send($message)
 }
`

