---
Author: anupam majhi
Publisher: 
Copyright: 
Email: 
Version: 1.0.0.0
Encoding: ascii
License: cc0
PoshCode ID: 5328
Published Date: 2016-07-24t15
Archived Date: 2016-11-07t08
---

# get-printerdetails - 

## Description

in your server environments sometimes to get the details of printers on a print server people need to log-in to the print server, open mmc console, go to print management, add servers and then get to see the printer details.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $ScriptBlock = {
                     function Compare_PrintServer
                     {
                     $PrintPaperSizeArray2 = @()
 
                         #$host.Runspace.ThreadOptions = "ReuseThread"
                         Add-Type -AssemblyName System.Printing
                         $Printer = gwmi Win32_Printer
                         $PrintersTCP = Get-WmiObject Win32_TcpIpPrinterPort
                         
                         $ps2 = New-Object System.Printing.LocalPrintServer
                         $allqueues2 = $ps2.GetPrintQueues() | %{
                            
                            if($_.name -ne "Microsoft XPS Document Writer"){
                                 $thequeue2name = $_.name
                                 $portname = $Printer | where {$_.name -eq $thequeue2name} | select -ExpandProperty portname
                                 $published = $Printer | where {$_.name -eq $thequeue2name} | select -ExpandProperty published
                                 $portIP = $PrintersTCP | where {$_.name -eq $portname} | select -ExpandProperty HostAddress
                                 $thequeue2 = New-Object System.Printing.PrintQueue $ps2,$_.name
                                 $thequeue2.DefaultPrintTicket.PageMediaSize.PageMediaSizeName
                                 $PrintPaperSizeArray2 += "$($thequeue2.Description),$($thequeue2.DefaultPrintTicket.PageMediaSize.PageMediaSizeName),$($thequeue2.DefaultPrintTicket.Duplexing),$($thequeue2.DefaultPrintTicket.outputcolor),$($thequeue2.DefaultPrintTicket.stapling),$portname,$PortIP,$Published"
                             }
                         }
                         
                         
                     $newPrintServDetails = $PrintPaperSizeArray2
                     "<h2 style='text-align:centre;color:green'>Server : $(hostname)</h2>">> $printerhtmlout
                                         
                     "<hr>"  >> $printerhtmlout
                       
                       
                     "<table border='1' cellpadding='4'><tr style='text-align:center;font-weight:bold;font-size:20px'><td>Print Queue</td><td>Driver Model</td><td>Location</td><td>Paper Size</td><td>Duplex</td><td>Color</td><td>Staple</td><td>Port</td><td>IP Address</td><td>Listed in AD</td></tr>" >> $printerhtmlout
 
                          $newPrintServDetails | %{
                             $newProperties = $_.split(",")
                                      
                                 "<tr>
                                 <td>$($newProperties[0])</td>"  >> $printerhtmlout
                                 
                                 
                                 
                                 
                                 
                                
                                
                                 
                                 
                                             
                                 
                                 "</tr>"  >> $printerhtmlout
                             }
                         
                         
 
                     "</table>" >> $printerhtmlout
                     }
 
 
                      $printerhtmlout = "D:\Temp\PrinterDetails.html"
                      if(Test-Path $printerhtmlout)
                         {
                         del $printerhtmlout
                         }
                      Compare_PrintServer
                      
                      $versionMismatch = ""
                     $onlyx64 = ""
                     $onlyx86 = ""
 
 
                     $Driverlist = gwmi -class win32_printerdriver -ComputerName .
 
                     foreach($driver in $Driverlist){
                         $drivername = $driver.name
                         $drivername = $drivername.Split(",")
                         $drivername1 = $drivername[0]
                         
                         if($driver.SupportedPlatform -eq "Windows x64"){
 
                         $x64drvpath = "HKLM:\System\CurrentControlSet\Control\print\Environments\Windows x64\Drivers\Version-3\" + $drivername[0]
                         $x86drvpath = "HKLM:\System\CurrentControlSet\Control\print\Environments\Windows NT x86\Drivers\Version-3\" + $drivername[0]
                         
                         $x64regpath = "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\print\Environments\Windows x64\Drivers\Version-3\" + $drivername[0]
 
                         $x86regpath = "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\print\Environments\Windows NT x86\Drivers\Version-3\" + $drivername[0]
 
                         
                             if(Test-Path $x64drvpath){
                         
                         	$x64ver = ""
                     		$x86ver = ""
                     		get-itemproperty -path Registry::$x64regpath | Select-Object -ExpandProperty driverversion | %{$x64ver += $_.ToString()}
                     		
                             		if(Test-Path $x86drvpath){
                                     		get-itemproperty -path Registry::$x86regpath | Select-Object -ExpandProperty driverversion | %{$x86ver += $_.ToString()}
                                     		
                                             if($x64ver -ne $x86ver){
                                     			$versionMismatch += "<p>$drivername1</p>"
                                     		}
                                     }
                             	   else{
                             			$onlyx64 += "<p>$drivername1</p>"
                             		}
 
                     	    }
                         }
                     	
                     	if($driver.SupportedPlatform -eq "Windows NT x86"){
                     		$x64drvpath = "HKLM:\System\CurrentControlSet\Control\print\Environments\Windows x64\Drivers\Version-3\" + $drivername[0]
                             $x86drvpath = "HKLM:\System\CurrentControlSet\Control\print\Environments\Windows NT x86\Drivers\Version-3\" + $drivername[0]
                         
                         $x64regpath = "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\print\Environments\Windows x64\Drivers\Version-3\" + $drivername[0]
 
                         $x86regpath = "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\print\Environments\Windows NT x86\Drivers\Version-3\" + $drivername[0]
                         
                             if(Test-Path $x64drvpath){
                     	    }
                     	   else{
                     		  $onlyx86 += "<p>$drivername1</p>"
                     	   }
 
                     	}
 
                     }
 
 
                     "<br><h2>Driver Version Mismatch : </h2>" >> $printerhtmlout
                     if($versionMismatch){
                     $versionMismatch >> $printerhtmlout
                     }
                     else{
                     "<h3>No Mismatch Found</h3>" >> $printerhtmlout
                     }
                     "<br><h2>Driver Version x86 not present - only x64 available : </h2>" >> $printerhtmlout
                     if($onlyx64){
                     $onlyx64 >> $printerhtmlout
                     }
                     else{
                     "<h3>No Mismatch Found</h3>" >> $printerhtmlout
                     }
                     "<br><h2>Driver Version only x86 available : </h2>" >> $printerhtmlout
                     if($onlyx86){
                     $onlyx86 >> $printerhtmlout
                     }
                     else{
                     "<h3>No Mismatch Found</h3>" >> $printerhtmlout
                     }
                     
 Get-Content $printerhtmlout
 }
 #---------------------------------------------------------------------------------------------------------------
 
 #$ScriptPath  = split-path $SCRIPT:MyInvocation.MyCommand.Path -parent
 
 Write-Host "----------------------------------------------------------------------"
 Write-Host "PRINT SERVER DETAILS - v 1.0.0.0`nDeveloped by - Anupam Majhi`nThePowershellKid" -ForegroundColor green
 Write-Host "----------------------------------------------------------------------`n`n"
 
 function MyPopup
 {
     Param($msg)
     $intAnswer = 7
     While ($intAnswer -eq 7)
     {
     $a = new-object -comobject wscript.shell
     $intAnswer = $a.popup($msg, 0,"Print Server Details - EXIS",4)
     }
 }
 
 Add-Type -AssemblyName Microsoft.VisualBasic
 $ServerName = [Microsoft.VisualBasic.Interaction]::InputBox("Enter Print Server Name To Query", "FETCH PRINT SERVER DETAILS", "Please enter SERVER name here")
 if($ServerName -ne ''){
 $mycreds = Get-Credential
 
 Write-Host "----------------------------------------------------------------------"
 Write-Host "Please wait while the Print Server details are fetched from $ServerName" -ForegroundColor yellow
 Write-Host "----------------------------------------------------------------------`n`n"
 Invoke-Command -ComputerName $ServerName -ScriptBlock $ScriptBlock -credential $mycreds  > "$env:userprofile\Desktop\$ServerName.html"
 "<hr><p><b>Executed on $(Get-Date) -- Executed by $($explorerdetails = (Get-WmiObject -Class Win32_Process | Where-Object {$_.ProcessName -eq 'explorer.exe'});
 $explorercount = $explorerdetails | Measure-Object | select -expandproperty count;
 if($explorercount -gt 1){$explorerdetails[0].GetOwner() | select -expandproperty User}
 elseif($explorercount -eq 1){$explorerdetails.GetOwner() | select -expandproperty User};) ------------</p></b>" >> "$env:userprofile\Desktop\$ServerName.html"
 
 if($error){
 Write-Host "----------------------------------------------------------------------"
 Write-Host "ERROR executing" -ForegroundColor red
 Write-Host "----------------------------------------------------------------------"
 MyPopup "Error in execution"
 $error >> "$env:userprofile\Desktop\$ServerName.html"
 Start-Process "$env:userprofile\Desktop\$ServerName.html"
 }
 else{
 Write-Host "----------------------------------------------------------------------"
 Write-Host "Printer Details Fetching COMPLETE!!!" -ForegroundColor green
 Write-Host "----------------------------------------------------------------------"
 
 MyPopup "Program Executed for $($ServerName)"
 
 Start-Process "$env:userprofile\Desktop\$ServerName.html"
 }
 
 }
`

