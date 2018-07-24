---
Author: archdeacon
Publisher: 
Copyright: 
Email: 
Version: 4.0
Encoding: ascii
License: cc0
PoshCode ID: 3157
Published Date: 2012-01-08t22
Archived Date: 2012-01-26t00
---

# gprs online log - 

## Description

scan the system event log for all gprs online activity – pcmcia, usb, mobile phone, etc. a balloon tip will also be issued when the sim card is about to expire.

## Comments



## Usage



## TODO



## function

`get-osversion`

## Code

`#
 #
 <#
 .SYNOPSIS
 Get-GprsTime (V4.0 Update for Windows 7 and allow time correction)
              (V4.4 'Interval' now incorporate previous 'FormatSpan' function)
              (V4.6 'Adjust' pattern will now reject times like '1:300:50')
 Check the total connect time of any GPRS SIM devices from a specified date. 
 Use 'Get-Help .\Get-GprsTime -full' to view Help for this script.
 .DESCRIPTION
 Display all the GPRS modem Event Log entries. While applications issued by the 
 mobile phone manufacturers will invariably monitor only their own usage, this
 will show any logged GPRS activity, be it via PCMCIA, USB, mobile phone, etc.
 Use the -Verbose switch for some extra information if desired. A default value
 can be set with the -Monthly switch but can be temporarily overridden with any 
 -Start value and deleted by entering an invalid date.  Now uses .NET Parse to 
 use any culture date input. Switches -M and -S cannot be used together.
    A Balloon prompt will be issued in the Notification area for the 5 days 
 before the nominal month end. 
 NOTE:  this can effectively be suppressed by using a value higher than the SIM
 card term, ie something like -Account 90 for a 30 day card which will override 
 the default setting. 
 Define a function in  $profile to set any permanent switches, for example to
 remove 1hour 15 minutes from the total each time.
 function GPRS {
    Invoke-Expression ".\Get-GprsTime -Adjust 1:15:00- $args" 
 }
 
 .EXAMPLE
 .\Get-GprsTime.ps1 -Monthly 4/8/2011
 
 This will set the default search date to start from 4/8/2011 and is used to
 reset the start date each month for the average 30/31 day SIM card.
 .EXAMPLE
 .\Get-GprsTime.ps1 -Start 12/07/2009 -Account 100 -Verbose
 
 Search from 12/07/2011 and also show (Verbose) details for each session. The
 Account switch will override any balloon prompt near the SIM expiry date.
 .EXAMPLE
 .\Get-GprsTime.ps1 5/9/2011 -v
 
 Search one day only and show session details.
 .EXAMPLE
 .\Get-GprsTime.ps1 -Today
 
 Show all sessions for today. This always defaults to verbose output.
 .EXAMPLE
 .\Get-GprsTime.ps1 -Debug
 
 This shows the first available Event Log record & confirms switch settings. 
 (Note that it will probably not be the first 'RemoteAccess' or 'RasClient'
 record).
 .EXAMPLE
 .\Get-GprsTime -Adjust 1:20:00
 .\Get-GprsTime -Hours  1:20:00
 
 If the same SIM card is used on another computer then such times will have
 to be added here manually (this should be put in a function in $profile so
 that it will automatically be included - see example above).
 To remove an amount of time, just put a '-' sign after the desired figure.
 The alias 'Hours' may be used to distinguish from the 'Account' parameter.
 .NOTES
 A shorter total time than that actually used will result if the Event Log
 does not contain earlier dates; just increase the log size in such a case.
 The author can be contacted at www.SeaStarDevelopment.Bravehost.com and a
 (binary compiled) Module version of this procedure is included in the file
 Gprs3xxx.zip download there; the execution time of the module being about 
 10 times faster than this script. 
 Use '(measure-Command {.\Get-GprsTime}).TotalSeconds' to confirm the times.
 For the Event Log to record connect & disconnect events, the modem software
 should be set to RAS rather than NDIS if possible.
 #>
 
 
 Param ([String] $start,
        [alias("HOURS")][string] $adjust,
        [String] $monthly,
        [Int]    $account = 0,     
        [Switch] $today,
        [Switch] $verbose,
        [Switch] $debug)
  
 Trap [System.Management.Automation.MethodInvocationException] {
     [Int]$line = $error[0].InvocationInfo.ScriptLineNumber
     [System.Media.SystemSounds]::Hand.Play()
     if ($line -eq  213) {
         Write-Warning "[$name] Current GPRS variable has been deleted."
         $monthly = ""
         [Environment]::SetEnvironmentVariable("GPRS",$monthly,"User")
     }
     else {
 	    Write-Warning "[$name] Date is missing or invalid $SCRIPT:form"
     }
     exit 1
 }
 function Get-OSVersion($computer,[ref]$osv) {
    $os = Get-WmiObject -class Win32_OperatingSystem -computerName $computer
    Switch -regex ($os.Version) {
      default             { $osv.value = "unknown" }
    }
 } 
 
 $osv = $null
 Get-OSVersion -computer 'localhost' -osv ([ref]$osv)
 if ($osv -eq 'xp') {
     $logname = 'System'
     $connEvent = 20158
     $discEvent = 20159
     $source = 'RemoteAccess'
 }
     $logname = 'Application'
     $connEvent = 20225
     $discEvent = 20226
     $source = 'RasClient'
 }
 $entryType = 'Information'
 $logEntry = $null
 $source = 'Undetermined'
              Sort-Object TimeGenerated |
                 Select-Object -first 1
 if ($oldest.TimeGenerated) { 
     $logEntry = "$logname Event records available from - $($oldest.TimeGenerated.ToLongDateString())"
 }
 $name = $myInvocation.MyCommand
 $newLine = "[$name] The switches -Start and -Monthly can only be used separately."
 if ($debug) {
     $DebugPreference = 'Continue'
 }
 if ($start -and $monthly) {
     [System.Media.SystemSounds]::Hand.Play()
     Write-Warning "$newLine"
     exit 1
 }
 $SCRIPT:form = $null
 
 if ($Host.CurrentCulture -eq $Host.CurrentUICulture) {
     $SCRIPT:form = '-Use format mm/dd/year'               
     if ($culture -eq 6) {
 		$SCRIPT:form = '-Use format dd/mm/year'
     }
 }
 if ($SCRIPT:form) {
    $debugForm = $SCRIPT:form.Replace('Use format','')
    Write-Debug "Current local system Date format is $debugForm"
 }
 $ErrorActionPreference = 'SilentlyContinue'
 $VerbosePreference = 'SilentlyContinue'
 $conn = $disc = $default = $print = $null              
 $timeNow = [DateTime]::Now 
 $insert = "since"
 if ($verbose) {
     $VerbosePreference = 'Continue'
 }
 
 function CreditMsg ($value, $value2) {
     $value = [Math]::Abs($value)
     $prefix = "CURRENT"
     [DateTime] $creditDT = $value2 
     $number = $creditDT - [DateTime] $thisDay
     [String] $credit = $creditDT   
     $credit = "{0:d}" -f [DateTime]$credit
     Switch($number.Days) {
         1            {$prefix = "($value days) will expire tomorrow"; break}
         0            {$prefix = "($value days) will expire today"; break}
         -1           {$prefix = "($value days) expired yesterday"; break}
         {($_ -lt 0)} {$prefix = "($value days) expired on $credit"; break}
         {($_ -le 5)} {$prefix = "($value days) will expire on $credit"}
     }
     return $prefix
 }
 
     Switch -regex ($value) {
         '^00?:00?:\d+(.*)$'    {$suffix = "seconds"; break}
         '^00?:\d+:\d+(.*)$'    {$suffix = "minutes"; break}
         '^\d+:\d+:\d+(.*)$'    {$suffix = "  hours"; break}
         '^(\d+)\.(\d+)(\D.*)$' {$suffix = "  hours"
                                  $pDays  = $matches[1]
                                  $pHours = $matches[2]
                                  [string]$pRest  = $matches[3]
                                  [string]$tHours = (([int]$pDays) * 24)+[int]$pHours
                                  $value  = $tHours + $pRest; break}
     }
     return "$value $suffix"
 }
 
 function CheckSetting ($value) {
 	   $print = $default.ToShortDateString() 
     }
     else {
 	   $print = "(Value not set)"
     }
     return $print
 }
 $getGprs = [Environment]::GetEnvironmentVariable("GPRS","User")
     $default = "{0:d}" -f [datetime]$getGprs 
     $default = [DateTime]::Parse($default)  
     $checkParts = "{0:yyyy},{0:%M}" -f $default
     $times = $checkParts.Split(',')
     if($account -eq 0) {
        $account = $dayCount
 	   $summary = "$($dayCount.ToString()) days" 
     }
     else {
 	   $summary = "$($account.Tostring()) days"
     }
     if ($text -ne "CURRENT") {
    	[void] [System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms")
     	$objNotifyIcon = New-Object System.Windows.Forms.NotifyIcon 
     	$objNotifyIcon.Icon = [System.Drawing.SystemIcons]::Exclamation 
     	$objNotifyIcon.BalloonTipIcon  = "Info" 
     	$objNotifyIcon.BalloonTipTitle = "GPRS online account"
     	$objNotifyIcon.BalloonTipText  = "Credit $text"
     	$objNotifyIcon.Visible = $True 
     	$objNotifyIcon.ShowBalloonTip(10000)
     }
 }
 else {
     $summary = "(Days not set)"
     if ((!$today) -and (!$monthly) -and (!$start)) {
 	   [System.Media.SystemSounds]::Hand.Play()
 	   Write-Warning("Monthly date is either invalid or not set.")
 	   exit 1
     }
 }
 if ($start) {
     [DateTime]$limit = $start
     $convert = "{0:D}" -f $limit
     $print = CheckSetting $default
 } 
 
 if ($monthly) {
     Write-Output "Setting GPRS (monthly) environment variable to: $monthly"
     $gprs = [String]$start.Replace('00:00:00','')
     [Environment]::SetEnvironmentVariable("GPRS",$gprs,"User")
     $convert = "{0:D}" -f $limit
     $print = $limit.ToShortDateString()
 }
 
 if ($today) {                    
     [DateTime] $limit = (Get-Date)
     $convert = "{0:D}" -f $limit
     $insert = "for today"
     $print = CheckSetting $default
 }
 if ((!$today) -and (!$monthly) -and (!$start)) {        
     if ($default) {
        $monthly = $default
 	   [DateTime] $limit = $monthly
 	   $convert = "{0:D}" -f $limit
 	   $print = CheckSetting $default
     }
 }
 
 if ($adjust) {
    $pattern = '^(\d{1,3}):([0-5]?[0-9]):([0-5]?[0-9])-?$'
    if ($adjust -notmatch $pattern ) { 
       Write-Warning "Invalid input ($adjust) - use <hours>:<minutes>:<seconds>" -WarningAction Continue
       return
    $adjustment = New-TimeSpan -hours $matches[1] -minutes $matches[2] -seconds $matches[3]
    $outString =  $adjust.Replace('-','')
    $outString = interval $outString
 }
 
 Write-Verbose "All records $($insert.Replace('for ','')) - $convert"
 Write-Verbose "Script activation - User [$($env:UserName)] Computer [$($env:ComputerName)]"
 Write-Output ""
 Write-Output "Calculating total connect time of all GPRS modem devices..."
 Write-Debug "Using - [Search date] $($limit.ToShortDateString()) [Account] $summary [GPRS Monthly] $print"
 if ($logEntry) {					
     Write-Debug "$logEntry"
 }
 
 $lines = Get-EventLog $logname | Where-Object {($_.TimeGenerated -ge $limit) -and `
     ($_.EventID -eq $discEvent -or $_.EventID -eq $connEvent)}  
 if ($lines) {
     Write-Verbose "A total of $([Math]::Truncate($lines.Count/2)) online sessions extracted from the $logname Event Log."
 }
 else {
     Write-Output "(There are no events indicated in the $logname Event Log)"
 }
 $lines | ForEach-Object {
     try {
        $source = $_.Source
     }
     catch {
        return
     }
        $disc = $_.TimeGenerated
     } 
        $conn = $_.TimeGenerated	
     if ($disc -ne $null -and $conn -ne $null -and $disc -gt $conn) {
        $diff = $disc - $conn
        $total += $diff
        $convDisc = "{0:G}" -f $disc
        $convConn = "{0:G}" -f $conn
        $period = interval $diff
        Write-Verbose "Disconnect at $convDisc. Online - $period"
        Write-Verbose "   Connect at $convConn."
     }
 Write-Verbose "Using local event source - $logname Event Log [$source]"
 $period = interval $total
 
 if ($adjustment -and ($total -match $pattern)) {
    $outDate = New-TimeSpan -hours $matches[1] -minutes $matches[2] -seconds $matches[3]
    if (!$adjust.EndsWith('-')) {
       Write-Debug "Processing (Timespan) string: $temp"
       $show = (interval $temp).Replace('   ',' ')
       Write-Output "Total online usage $insert $convert is $show"
       Write-Output "(including $($outString.Replace('   ',' ')) adjustment time)"
    }
    else {
       if ($outDate.TotalSeconds -gt $adjustment.TotalSeconds) {
          $period = $outDate - $adjustment
          $temp = [String]$period
          Write-Debug "Processing (Timespan) string: $temp"
          $show = (interval $temp).Replace('   ',' ')
          Write-Output "Total online usage $insert $convert is $show"
          Write-Output "(excluding $($outString.Replace('   ',' ')) adjustment time)"
       }
       else {
          $adjust = $adjust.Replace('-','')
          Write-Output "Total online usage $insert $convert is $($period.ToString().Replace('   ',' '))."
          Write-Warning "Total usage exceeded ($adjust) - no adjustment performed"
       }
    }
 }
 else {
    Write-OutPut "Total online usage $insert $convert is $($period.ToString().Replace('   ',' '))."
 }
 Write-Output ""
`

