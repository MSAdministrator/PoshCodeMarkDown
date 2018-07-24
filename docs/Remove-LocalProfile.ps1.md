---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 6.0
Encoding: ascii
License: cc0
PoshCode ID: 6842
Published Date: 2017-04-14t18
Archived Date: 2017-05-22t04
---

# remove-localprofile - 

## Description

this is script will first ask for a computername and then will scan the win32_userprofile wmi class and present the user with all of the possible user profiles to remove. after the profile has been deleted, the user has a choice to continue to remove another profile or quit. this script will only work against vista and above client os’s and window 2008 and above server os’s, but can be ran from any os that has powershell installed. updated to include windows 10 and server 2016.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 .SYNOPSIS   
     Interactive menu that allows a user to connect to a local or remote computer and remove a local profile. 
 .DESCRIPTION 
     Presents an interactive menu for user to first make a connection to a remote or local machine.  After making connection to the machine,  
     the user is presented with all of the local profiles and then is asked to make a selection of which profile to delete. This is only valid 
     on Windows Vista OS and above for clients and Windows 2008 and above for server OS.    
 .NOTES   
     Name: Remove-LocalProfile 
     Author: Boe Prox 
     DateCreated: 26JAN2011       
 .LINK   
     http://boeprox.wordpress.com
     http://msdn.microsoft.com/en-us/library/ee886409%28v=vs.85%29.aspx 
 .EXAMPLE  
 Remove-LocalProfile 
  
 Description 
 ----------- 
 Presents a text based menu for the user to interactively remove a local profile on local or remote machine.    
 #>  
  
 $computer = Read-Host "Please enter a computer name" 
 If ($computer -ne $Env:Computername) { 
     If (!(Test-Connection -comp $computer -count 1 -quiet)) { 
         Write-Warning "$computer is not accessible, please try a different computer or verify it is powered on." 
         Break 
         } 
     } 
 Try {     
     If ((Get-WmiObject -ComputerName $computer Win32_OperatingSystem -ea stop).Version -lt ("0:D2" -f 6.0)) { 
         Write-Warning "The Operating System of the computer is not supported.`nClient: Vista and above`nServer: Windows 2008 and above." 
         Break 
         } 
     } 
 Catch { 
     Write-Warning "$($error[0])" 
     Break 
     }     
 Do {     
 Try { 
     $users = Get-WmiObject -ComputerName $computer Win32_UserProfile -filter "LocalPath Like 'C:\\Users\\%'" -ea stop 
     } 
 Catch { 
     Write-Warning "$($error[0]) " 
     Break 
     }     
 $num_users = $users.count 
  
 Write-Host -ForegroundColor Green "User profiles on $($computer):" 
  
     For ($i=0;$i -lt $num_users; $i++) { 
         Write-Host -ForegroundColor Green "$($i): $(($users[$i].localpath).replace('C:\Users\',''))" 
         } 
     Write-Host -ForegroundColor Green "q: Quit" 
     Do {     
         $account = Read-Host "Select a number to delete local profile or 'q' to quit" 
         If ($account -NotLike "q*") { 
             $account = $account -as [int] 
             } 
         }         
     Until (($account -lt $num_users -AND $account -match "\d") -OR $account -Like "q*") 
     If ($account -Like "q*") { 
         Break 
         } 
     Write-Host -ForegroundColor Yellow "Deleting profile: $(($users[$account].localpath).replace('C:\Users\',''))" 
     ($users[$account]).Delete() 
     Write-Host -ForegroundColor Green "Profile:  $(($users[$account].localpath).replace('C:\Users\','')) has been deleted" 
  
     $yes = New-Object System.Management.Automation.Host.ChoiceDescription "&Yes","Remove another profile." 
  
     $no = New-Object System.Management.Automation.Host.ChoiceDescription "&No","Quit profile removal" 
  
     $choice = [System.Management.Automation.Host.ChoiceDescription[]] @($yes,$no) 
  
     [int]$default = 0 
  
     $userchoice = $host.ui.PromptforChoice("Information","Remove Another Profile?",$choice,$default) 
     } 
 Until ($userchoice -eq 1)
`

