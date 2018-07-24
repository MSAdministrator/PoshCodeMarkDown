---
Author: jan egil ring
Publisher: 
Copyright: 
Email: jan.egil.ring@powershell.no
Version: 1.1
Encoding: utf-8
License: cc0
PoshCode ID: 5658
Published Date: 2017-01-01t00
Archived Date: 2017-05-18t00
---

# invoke-windowsupdate - invoke-windowsupdate.ps1

## Description

my revised version of jan egil ring’s script, with the corrections noted by the commenters in his blog article.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ###########################################################################"
 #
 #
 #
 #
 #
 #
 ###########################################################################"
 
 
 $EmailReport = $true
 $FileReport = $false
 $To = "itops@domain.com"
 $From = "WindowsUpdateScript@domain.com"
 $SMTPServer = "smtp.domain.com"
 $FileReportPath = "C:\WindowsUpdate\"
 $AutoRestart = $true
 $AutoRestartIfPending = $true
 
 $Path = $FileReportPath + "$env:ComputerName" + "_" + (Get-Date -Format dd-MM-yyyy_HH-mm).ToString() + ".html"
 
 if (Test-Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired"){
 
 if ($EmailReport -eq $true) {
 $pendingboot = @{$false="was pending for a restart from an earlier Windows Update session. Due to the reboot preferences in the script, a reboot was not initiated."; $true="was restarted due to a pending restart from an earlier Windows Update session."}
 $status = $pendingboot[$AutoRestartIfPending]
  $messageParameters = @{                        
                 Subject = "Windows Update report for $env:ComputerName.$env:USERDNSDOMAIN - $((Get-Date).ToShortDateString())"                        
                 Body = "Invoke-WindowsUpdate was run on $env:ComputerName, and the server $status `nPlease run Invoke-WindowsUpdate again when the server is rebooted."               
                 from = $From                        
                 To = $To                      
                 SmtpServer = $SMTPServer                         
             }                        
             Send-MailMessage @messageParameters -BodyAsHtml
 
 if ($FileReport -eq $true) {
 "Invoke-WindowsUpdate was run on $env:ComputerName, and the server $status `nPlease run Invoke-WindowsUpdate again when the server is rebooted." | Out-File -FilePath $path
 }
 
 if ($AutoRestartIfPending) {shutdown.exe /t 0 /r }  }
 exit
 			
 }
 
 $updateSession = new-object -com "Microsoft.Update.Session"
 write-progress -Activity "Updating" -Status "Checking available updates"   
 $updates=$updateSession.CreateupdateSearcher().Search($criteria).Updates
 $downloader = $updateSession.CreateUpdateDownloader()          
 $downloader.Updates = $Updates
 
 if ($downloader.Updates.Count -eq "0") {
 
 if ($EmailReport -eq $true) {
  $messageParameters = @{                        
                 Subject = "Windows Update report for $env:ComputerName.$env:USERDNSDOMAIN - $((Get-Date).ToShortDateString())"                        
                 Body = "Invoke-WindowsUpdate was run on $env:ComputerName, but no new updates were found. Please try again later."               
                 from = $From                        
                 To = $To                      
                 SmtpServer = $SMTPServer                         
             }                        
             Send-MailMessage @messageParameters -BodyAsHtml
 			}
 			
 if ($FileReport -eq $true) {
 "Invoke-WindowsUpdate was run on $env:ComputerName, but no new updates were found. Please try again later." | Out-File -FilePath $Path
 }
 
 }
 else
 {
 write-progress -Activity 'Updating' -Status "Downloading $($downloader.Updates.count) updates"  
 
 $Criteria="IsInstalled=0 and Type='Software'" 
 $resultcode= @{0="Not Started"; 1="In Progress"; 2="Succeeded"; 3="Succeeded With Errors"; 4="Failed" ; 5="Aborted" }
 $Result= $downloader.Download()
 
 if (($Result.Hresult -eq 0) ñand (($result.resultCode ñeq 2) -or ($result.resultCode ñeq 3)) ) {
        $updatesToInstall = New-object -com "Microsoft.Update.UpdateColl"
        $Updates | where {$_.isdownloaded} | foreach-Object {$updatesToInstall.Add($_) | out-null 
 }
 
 $installer = $updateSession.CreateUpdateInstaller()       
 $installer.Updates = $updatesToInstall
 
 write-progress -Activity 'Updating' -Status "Installing $($Installer.Updates.count) updates"        
 
 $installationResult = $installer.Install()        
 $Global:counter=0       
 
 #$Report = $installer.updates | 
 
 $Report = $installer.updates |
         Select-Object -property Title,EulaAccepted,@{Name=íResultí;expression={$ResultCode[$installationResult.GetUpdateResult($Global:Counter).resultCode ] }},@{Name=íReboot requiredí;expression={$installationResult.GetUpdateResult($Global:Counter++).RebootRequired }} |
 
 
 if ($EmailReport -eq $true) {
  $messageParameters = @{                        
                 Subject = "Windows Update report for $env:ComputerName.$env:USERDNSDOMAIN - $((Get-Date).ToShortDateString())"                        
                 Body =  $Report | Out-String                 
                 from = $From                        
                 To = $To                      
                 SmtpServer = $SMTPServer                         
             }                        
             Send-MailMessage @messageParameters -BodyAsHtml
 			}
 
 if ($FileReport -eq $true) {
 $Report | Out-File -FilePath $path
 }
 
 if ($autoRestart -and $installationResult.rebootRequired) { shutdown.exe /t 0 /r }       
 }
 }
`

