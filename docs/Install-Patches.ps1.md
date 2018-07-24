---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2689
Published Date: 2011-05-20t20
Archived Date: 2011-05-27t22
---

# install-patches - 

## Description

this module uses psexec,vbscript and powershell to install patches on local or remote systems. this does require psexec to be in the same directory as where you are running the function within this module. save as a .psm1 file.

## Comments



## Usage



## TODO



## module

`create-updatevbs`

## Code

`#
 #
 <#
 Save this with a .PSM1 extension and use Import-Module to properly load the functions.
 ie: Install-Patches.psm1
 Import-Module Install-Patches.psm1
 #>
 Write-Verbose "Checking Administrator credentials"
 If (-NOT ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole(`
     [Security.Principal.WindowsBuiltInRole] "Administrator"))
 {
     Write-Warning "You are not running this as an Administrator!`nPlease re-run this with an Administrator account."
     Break
 }
 
 Function Create-UpdateVBS {
 Param ($computername)
 $vbsstring = @"
 ON ERROR RESUME NEXT
 CONST ForAppending = 8
 CONST ForWriting = 2
 CONST ForReading = 1
 strlocalhost = "."
 Set oShell = CreateObject("WScript.Shell") 
 set ofso = createobject("scripting.filesystemobject")
 Set updateSession = CreateObject("Microsoft.Update.Session")
 Set updateSearcher = updateSession.CreateupdateSearcher()
 Set updatesToInstall = CreateObject("Microsoft.Update.UpdateColl")
 Set searchResult = updateSearcher.Search("IsInstalled=0 and Type='Software'")
 Set objWMI = GetObject("winmgmts:\\" & strlocalhost & "\root\CIMV2")
 set colitems = objWMI.ExecQuery("SELECT Name FROM Win32_ComputerSystem")
 	For Each objcol in colitems
 		strcomputer = objcol.Name
 	Next
 set objtextfile = ofso.createtextfile("C:\" & strcomputer & "_patchlog.csv", True)
 objtextfile.writeline "Computer,Title,KB,InstallResultCode"
 If searchresult.updates.count = 0 Then
 	Wscript.echo "No updates to install."
 	objtextfile.writeline strcomputer & ",NA,NA,NA"
 	Wscript.Quit
 Else
 For I = 0 To searchResult.Updates.Count-1
     set update = searchResult.Updates.Item(I)
 	    If update.IsDownloaded = true Then
 	       updatesToInstall.Add(update)	
 	End If
 Next
 End If
 err.clear
 Wscript.Echo "Installing Updates"
 Set installer = updateSession.CreateUpdateInstaller()
 installer.Updates = updatesToInstall
 Set installationResult = installer.Install()
 	If err.number <> 0 Then
 		objtextfile.writeline strcomputer & "," & update.Title & ",NA," & err.number
 	Else		
 		For I = 0 to updatesToInstall.Count - 1
 		objtextfile.writeline strcomputer & "," & updatesToInstall.Item(i).Title & ",NA," & installationResult.GetUpdateResult(i).ResultCode 
 		Next
 	End If
 Wscript.Quit
 "@
 
 Write-Verbose "Creating vbscript file on $computer"
 Try {
     $vbsstring | Out-File "\\$computername\c$\update.vbs" -Force
     Return $True
     }
 Catch {
     Write-Warning "Unable to create update.vbs!"
     Return $False
     }
 }
 
 
 Function Format-InstallPatchLog {
     [cmdletbinding()]
     param ($computername)
     
     $installreport = @()
     If (Test-Path "\\$computername\c$\$($computername)_patchlog.csv") {
         [array]$CSVreport = Import-Csv "\\$computername\c$\$($computername)_patchlog.csv"
         If ($csvreport[0].title -ne "NA") {
             ForEach ($log in $CSVreport) {
                 $temp = "" | Select Computer,Title,KB,InstallResultCode
                 $temp.Computer = $log.Computer
                 $temp.Title = $log.title.split('\(')[0]
                 $temp.KB = ($log.title.split('\(')[1]).split('\)')[0]
                 Switch ($log.InstallResultCode) {
                     1 {$temp.InstallResultCode = "No Reboot required"}
                     2 {$temp.InstallResultCode = "Reboot Required"}
                     4 {$temp.InstallResultCode = "Failed to Install Patch"}
                     "-2145124316" {$temp.InstallResultCode = "Update is not available to install"}
                     Default {$temp.InstallResultCode = "Unable to determine Result Code"}            
                     }
                 $installreport += $temp
                 }
             }
         Else {
             $temp = "" | Select Computer, Title, KB,InstallResultCode
             $temp.Computer = $computername
             $temp.Title = "NA"
             $temp.KB = "NA"
             $temp.InstallResultCode = "NA"  
             $installreport += $temp            
             }
         }
     Else {
         $temp = "" | Select Computer, Title, KB,InstallResultCode
         $temp.Computer = $computername
         $temp.Title = "NA"
         $temp.KB = "NA"
         $temp.InstallResultCode = "NA"  
         $installreport += $temp      
         }
     Write-Output $installreport
 }
 
 Function Install-Patches {
 .SYNOPSIS    
     Install patches on a local or remote computer and generates a report.
 .DESCRIPTION  
     Install patches on a local or remote computer and generates a report with status of installation.
 .PARAMETER Computername  
     Name of the computer to install patches on.           
 .NOTES    
     Name: Install-Patches 
     Author: Boe Prox  
     DateCreated: 19May2011   
         
 .LINK    
     https://boeprox.wordpress.com  
 .EXAMPLE    
     Install-Patches -Computername Server1
     
     Description
     -----------
     Installs patches on Server1 and displays report with installation status.
 
 .EXAMPLE    
     Install-Patches -Computername Server1,Server2,Server3
     
     Description
     -----------
     Installs patches on Server1,Server2 and Server3 and displays report with installation status.    
 #>
 [cmdletbinding()]
 Param(
     [Parameter(Mandatory = $True,Position = 0,ValueFromPipeline = $True)]  
     [string[]]$Computername
     )
 Begin {
     If (-Not (Test-Path psexec.exe)) {
         Write-Warning "PSExec not in same directory as script!"  
         Break
         }
     }
 Process {
     ForEach ($computer in $computername) {
         If ((Test-Connection -ComputerName $computer -Count 1 -Quiet)) {
             Write-Verbose "Creating update.vbs file on remote server."
             If (Create-UpdateVBS -computer $computer) {
                 Write-Verbose "Patching computer: $($computer)"
                 .\psexec.exe -accepteula -s -i \\$computer cscript.exe C:\update.vbs
                 If ($LASTEXITCODE -eq 0) {
                     Write-Verbose "Successful run of install script!"
                     Write-Verbose "Formatting log file and adding to report"
                     Format-InstallPatchLog -computer $computer
                     }            
                 Else {
                     Write-Warning "Unsuccessful run of install script!"
                     $report = "" | Select Computer,Title,KB,IsDownloaded,InstallResultCode
                     $report.Computer = $computer
                     $report.Title = "ERROR"
                     $report.KB = "ERROR"
                     $report.IsDownloaded = "ERROR"
                     $report.InstallResultCode = "ERROR" 
                     Write-Output $report
                     }
                 }
             Else {
                 Write-Warning "Unable to install patches on $computer"
                 $report = "" | Select Computer,Title,KB,IsDownloaded,InstallResultCode
                 $report.Computer = $computer
                 $report.Title = "ERROR"
                 $report.KB = "ERROR"
                 $report.IsDownloaded = "ERROR"
                 $report.InstallResultCode = "ERROR" 
                 Write-Output $report              
                 }
             }
         Else {
             Write-Warning "$($Computer): Unable to connect!"
             } 
         } 
     }   
 }
 Export-ModuleMember Install-Patches
`

