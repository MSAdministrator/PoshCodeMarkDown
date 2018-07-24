---
Author: francois fournier
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6327
Published Date: 2016-04-26t19
Archived Date: 2016-04-30t00
---

# outlook pst backup - 

## Description

this script will make a copy of the local pst to the one drive folder

## Comments



## Usage



## TODO



## script

`log-start`

## Code

`#
 #
 
 <#
 
 .SYNOPSIS
 
   OutlookBackup
 
 
 .DESCRIPTION
 
   This script will make a copy of the local PST to the One drive folder
   The script will stop Outlook
   The script will Then copy the local PST to a backup directory.
   Outlook will the be restarted.
   
 
 .PARAMETER <Parameter_Name>
 
     No parameters are required
 
 
 .INPUTS
 
   None
 
 
 .OUTPUTS
 
   Log file in specified directory
   Email message
 
 .NOTES
 
   Version:        1.0
 
   Author:         Francois Fournier
 
   Creation Date:  22/04/2016
 
   Purpose/Change: Initial script development
 
   
   History
   Date			Author				Version		Comments
   22 Apr 2016	Francois Fournier	1.0		initial release
 
  
 
 .EXAMPLE
 
   Outlook Backup.ps1
 
 #>
 
 
 Function Log-Start{
   <#
   .SYNOPSIS
     Creates log file
 
   .DESCRIPTION
     Creates log file with path and name that is passed. Checks if log file exists, and if it does deletes it and creates a new one.
     Once created, writes initial logging data
 
   .PARAMETER LogPath
     Mandatory. Path of where log is to be created. Example: C:\Windows\Temp
 
   .PARAMETER LogName
     Mandatory. Name of log file to be created. Example: TapeReportEject.log
       
   .PARAMETER ScriptVersion
     Mandatory. Version of the running script which will be written in the log. Example: 1.5
 
   .INPUTS
     Parameters above
 
   .OUTPUTS
     Log file created
 
   .NOTES
     Version:        1.0
     Author:         Luca Sturlese
     Creation Date:  10/05/12
     Purpose/Change: Initial function development
 
     Version:        1.1
     Author:         Luca Sturlese
     Creation Date:  19/05/12
     Purpose/Change: Added debug mode support
 
   .EXAMPLE
     Log-Start -LogPath "C:\Windows\Temp" -LogName "TapeReportEject.log" -ScriptVersion "1.5"
   #>
     
   [CmdletBinding()]
   
   Param ([Parameter(Mandatory=$true)][string]$LogPath, [Parameter(Mandatory=$true)][string]$LogName, [Parameter(Mandatory=$true)][string]$ScriptVersion)
   
   Process{
     $sFullPath = $LogPath + "\" + $LogName
     
     
     If((Test-Path -Path $sFullPath)){
     }
     else {
     New-Item -Path $LogPath -Name $LogName -ItemType File
     
     }
     
     
     Add-Content -Path $sFullPath -Value "***************************************************************************************************"
     Add-Content -Path $sFullPath -Value "Started processing at [$([DateTime]::Now)]."
     Add-Content -Path $sFullPath -Value "***************************************************************************************************"
     Add-Content -Path $sFullPath -Value ""
     Add-Content -Path $sFullPath -Value "Running script version [$ScriptVersion]."
     Add-Content -Path $sFullPath -Value ""
     Add-Content -Path $sFullPath -Value "***************************************************************************************************"
     Add-Content -Path $sFullPath -Value ""
   
     Write-Debug "***************************************************************************************************"
     Write-Debug "Started processing at [$([DateTime]::Now)]."
     Write-Debug "***************************************************************************************************"
     Write-Debug ""
     Write-Debug "Running script version [$ScriptVersion]."
     Write-Debug ""
     Write-Debug "***************************************************************************************************"
     Write-Debug ""
   }
 }
  
 Function Log-Write{
   <#
   .SYNOPSIS
     Writes to a log file
 
   .DESCRIPTION
     Appends a new line to the end of the specified log file
   
   .PARAMETER LogPath
     Mandatory. Full path of the log file you want to write to. Example: C:\Windows\Temp\TapeReportEject.log
   
   .PARAMETER LineValue
     Mandatory. The string that you want to write to the log
       
   .INPUTS
     Parameters above
 
   .OUTPUTS
     None
 
   .NOTES
     Version:        1.0
     Author:         Luca Sturlese
     Creation Date:  10/05/12
     Purpose/Change: Initial function development
   
     Version:        1.1
     Author:         Luca Sturlese
     Creation Date:  19/05/12
     Purpose/Change: Added debug mode support
 
   .EXAMPLE
     Log-Write -LogPath "C:\Windows\Temp\TapeReportEject.log" -LineValue "This is a new line which I am appending to the end of the log file."
   #>
   
   [CmdletBinding()]
   
   Param ([Parameter(Mandatory=$true)][string]$LogPath, [Parameter(Mandatory=$true)][string]$LineValue)
   
   Process{
     Add-Content -Path $LogPath -Value $LineValue
   
     Write-Debug $LineValue
   }
 }
  
 Function Log-Error{
   <#
   .SYNOPSIS
     Writes an error to a log file
 
   .DESCRIPTION
     Writes the passed error to a new line at the end of the specified log file
   
   .PARAMETER LogPath
     Mandatory. Full path of the log file you want to write to. Example: C:\Windows\Temp\TapeReportEject.log
   
   .PARAMETER ErrorDesc
     Mandatory. The description of the error you want to pass (use $_.Exception)
   
   .PARAMETER ExitGracefully
     Mandatory. Boolean. If set to True, runs Log-Finish and then exits script
 
   .INPUTS
     Parameters above
 
   .OUTPUTS
     None
 
   .NOTES
     Version:        1.0
     Author:         Luca Sturlese
     Creation Date:  10/05/12
     Purpose/Change: Initial function development
     
     Version:        1.1
     Author:         Luca Sturlese
     Creation Date:  19/05/12
     Purpose/Change: Added debug mode support. Added -ExitGracefully parameter functionality
 
   .EXAMPLE
     Log-Error -LogPath "C:\Windows\Temp\TapeReportEject.log" -ErrorDesc $_.Exception -ExitGracefully $True
   #>
   
   [CmdletBinding()]
   
   Param ([Parameter(Mandatory=$true)][string]$LogPath, [Parameter(Mandatory=$true)][string]$ErrorDesc, [Parameter(Mandatory=$true)][boolean]$ExitGracefully)
   
   Process{
     Add-Content -Path $LogPath -Value "Error: An error has occurred [$ErrorDesc]."
   
     Write-Debug "Error: An error has occurred [$ErrorDesc]."
     
     If ($ExitGracefully -eq $True){
       Log-Finish -LogPath $LogPath
       Break
     }
   }
 }
  
 Function Log-Finish{
   <#
   .SYNOPSIS
     Write closing logging data & exit
 
   .DESCRIPTION
     Writes finishing logging data to specified log and then exits the calling script
   
   .PARAMETER LogPath
     Mandatory. Full path of the log file you want to write finishing data to. Example: C:\Windows\Temp\TapeReportEject.log
 
   .PARAMETER NoExit
     Optional. If this is set to True, then the function will not exit the calling script, so that further execution can occur
   
   .INPUTS
     Parameters above
 
   .OUTPUTS
     None
 
   .NOTES
     Version:        1.0
     Author:         Luca Sturlese
     Creation Date:  10/05/12
     Purpose/Change: Initial function development
     
     Version:        1.1
     Author:         Luca Sturlese
     Creation Date:  19/05/12
     Purpose/Change: Added debug mode support
   
     Version:        1.2
     Author:         Luca Sturlese
     Creation Date:  01/08/12
     Purpose/Change: Added option to not exit calling script if required (via optional parameter)
 
   .EXAMPLE
     Log-Finish -LogPath "C:\Windows\Temp\TapeReportEject.log"
 
 .EXAMPLE
     Log-Finish -LogPath "C:\Windows\Temp\TapeReportEject.log" -NoExit $True
   #>
   
   [CmdletBinding()]
   
   Param ([Parameter(Mandatory=$true)][string]$LogPath, [Parameter(Mandatory=$false)][string]$NoExit)
   
   Process{
     Add-Content -Path $LogPath -Value ""
     Add-Content -Path $LogPath -Value "***************************************************************************************************"
     Add-Content -Path $LogPath -Value "Finished processing at [$([DateTime]::Now)]."
     Add-Content -Path $LogPath -Value "***************************************************************************************************"
   
     Write-Debug ""
     Write-Debug "***************************************************************************************************"
     Write-Debug "Finished processing at [$([DateTime]::Now)]."
     Write-Debug "***************************************************************************************************"
   
     If(!($NoExit) -or ($NoExit -eq $False)){
       Exit
     }    
   }
 }
  
 Function Log-Email{
   <#
   .SYNOPSIS
     Emails log file to list of recipients
 
   .DESCRIPTION
     Emails the contents of the specified log file to a list of recipients
   
   .PARAMETER LogPath
     Mandatory. Full path of the log file you want to email. Example: C:\Windows\Temp\TapeReportEject.log
   
   .PARAMETER EmailFrom
     Mandatory. The email addresses of who you want to send the email from. Example: "admin@9to5IT.com"
 
   .PARAMETER EmailTo
     Mandatory. The email addresses of where to send the email to. Seperate multiple emails by ",". Example: "admin@9to5IT.com, test@test.com"
   
   .PARAMETER EmailSubject
     Mandatory. The subject of the email you want to send. Example: "Cool Script - [" + (Get-Date).ToShortDateString() + "]"
 
   .INPUTS
     Parameters above
 
   .OUTPUTS
     Email sent to the list of addresses specified
 
   .NOTES
     Version:        1.0
     Author:         Luca Sturlese
     Creation Date:  05.10.12
     Purpose/Change: Initial function development
 
   .EXAMPLE
     Log-Email -LogPath "C:\Windows\Temp\TapeReportEject.log" -EmailFrom "admin@9to5IT.com" -EmailTo "admin@9to5IT.com, test@test.com" -EmailSubject "Cool Script - [" + (Get-Date).ToShortDateString() + "]"
   #>
   
   [CmdletBinding()]
   
   Param ([Parameter(Mandatory=$true)][string]$LogPath, [Parameter(Mandatory=$true)][string]$EmailFrom, [Parameter(Mandatory=$true)][string]$EmailTo, [Parameter(Mandatory=$true)][string]$EmailSubject)
   
   Process{
     Try{
       $sBody = (Get-Content $LogPath | out-string)
       
       $sSmtpServer = "smtp.yourserver"
       $oSmtp = new-object Net.Mail.SmtpClient($sSmtpServer)
       $oSmtp.Send($EmailFrom, $EmailTo, $EmailSubject, $sBody)
       Exit 0
     }
     
     Catch{
       Exit 1
     } 
   }
 }
 
 function Retry-Command 
  { 
      param ( 
      [Parameter(Mandatory=$true)][string]$command,   
      [Parameter(Mandatory=$false)][int]$retries = 5,  
      [Parameter(Mandatory=$false)][int]$secondsDelay = 2 
      ) 
       
      $ErrorAction = "Stop" 
       
      $retrycount = 0 
      $completed = $false 
  
      while (-not $completed) { 
          try { 
              & $command 
              Write-Verbose ("Command [{0}] succeeded." -f $command) 
              Log-Write -LogPath $logFile -LineValue ("Command [{0}] succeeded." -f $command)
              $completed = $true 
          } catch { 
              if ($retrycount -ge $retries) { 
                  Write-Verbose ("Command [{0}] failed the maximum number of {1} times." -f $command, $retrycount) 
                  Log-Write -LogPath $logFile -LineValue ("Command [{0}] failed the maximum number of {1} times." -f $command, $retrycount)
                  throw 
              } else { 
                  Write-Verbose ("Command [{0}] failed. Retrying in {1} seconds." -f $command, $secondsDelay) 
                  Log-Write -LogPath $logFile -LineValue ("Command [{0}] failed. Retrying in {1} seconds." -f $command, $secondsDelay) 
                  Start-Sleep $secondsDelay 
                  $retrycount++ 
              } 
          } 
      } 
  } 
 
 
 $Now = Get-Date -uformat "%Y-%m-%d %Hh%M" 
 $Source = "C:\Source\" 
 $Destination = "C:\Destination\"
 $LogName = "OutlookBackup.log"
 $logFile = "C:\LogDir\OutlookBackup.log"
 
 
 Log-Start -LogPath $Destination -LogName $LogName -ScriptVersion "1.5"
 Log-Write -LogPath $logFile -LineValue "Stopping Outlook."
 get-process outlook* | stop-process | wait-process
 sleep 5
 
 Log-Write -LogPath $logFile -LineValue "Creating Folder $Destination$Now"
 New-Item -ItemType directory -Path "$Destination$Now" 
 
 Log-Write -LogPath $logFile -LineValue "Copying PSTs from $source"
 
 $Files = Get-Childitem $Source -Recurse 
 foreach ($File in $Files) { 
     if ($File -ne $NULL) { 
         Log-Write -LogPath $logFile -LineValue "Copying $($File.FullName) to $Destination$Now"
         Copy-Item $File.FullName -destination $Destination$Now -force | out-null 
 		} 
     else 
         { 
         Log-Write -LogPath $logFile -LineValue "No more files in $Source to copy."
         Write-Host "No more files in $Source to copy." -foregroundcolor "DarkRed" 
         } 
 }  
 sleep 5
 Log-Write -LogPath $logFile -LineValue "Lauching Outlook."
 
 
 Retry-Command -Command 'C:\Program Files\Microsoft Office\Root\Office16\OUTLOOK.EXE'
 
 Log-Write -LogPath $logFile -LineValue "Done."
 Log-Finish -LogPath $LogFile -NoExit $True
`

