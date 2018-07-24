---
Author: dvsbobloblaw
Publisher: 
Copyright: 
Email: 
Version: 5.2
Encoding: ascii
License: cc0
PoshCode ID: 4107
Published Date: 2013-04-15t18
Archived Date: 2013-05-09t12
---

# get-lastbootuptime - 

## Description

script to get the lastbootuptime of a computer.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <#
 .SYNOPSIS
 Gets the start time of the PC
 
 .DESCRIPTION
 The Get-LastBootUpTime function returns an LBUT object that represents the 
 time the computer started. You can use Get-LabstBootUpTime to generate the up 
 time of a PC. By default it uses EventID 6005 from the System log to determine 
 when the PC started. You can also specify to use PSRemoting or WMI commands 
 instead.
 
 .PARAMETER ComputerName
 Vaild input is the name of a computer or a computer object.
 
 .PARAMETER WMI
 Use WMI commands instead of event log.
 
 .PARAMETER Remote
 Will use Invoke-Command instead of -Computer parameter.
 
 .EXAMPLE
 Get-LastBootUpTime beefy-pc
 
 The command returns the LastBootUpTime of beefy-pc.
 
 .EXAMPLE
 (Get-ADComputer computer),'anothercomputer' | Get-LastBootUpTime.ps1 -Progress
 
 This command gets the LastBootUpTime of "computer" and "anothercomputer." 
 Showing progress up top.
 
 Name                           Value
 ----                           -----
 Name                           computer
 UpTime                         00:33:04.5693916
 LastBootUpTime                 4/12/2012 12:23:39 AM
 Name                           anothercomputer
 UpTime                         2.12:59:59.0814428
 LastBootUpTime                 4/17/2011 11:56:45 AM
 
 .EXAMPLE 
 Get-LastBootUpTime -ComputerName 'Jeff','Don','June',(Get-ADComputer 'Ed') -verbose -Progress -WMI
 
 This command uses the Get-WMI cmdlet instead of Get-EventLog to retrieve the last boot up time.
 
 .LINK
 http://blogs.msdn.com/b/dmuscett/archive/2009/05/27/get_2d00_wmicustom.aspx
 http://blogs.msdn.com/b/powershell/archive/2009/08/12/get-systemuptime-and-working-with-the-wmi-date-format.aspx?Redirected=true
 http://www.microsoft.com/technet/support/ee/transform.aspx?ProdName=Windows+Operating+System&ProdVer=5.2&EvtID=6005&EvtSrc=EventLog&LCID=1033
 
 .NOTES
 It may be undesirable to query the WMI if errors occur or the timeout variable.
 Author: DVSBOBLOBLAW
 Date: 4:15:2013
 Twitter: https://twitter.com/dvsbobloblaw
 
 .INPUTS
 An array of strings, an array of computer objects (any object that has a name parameter), or a mixture of both.
 
 
 .OUTPUTS
 An array of custom PSObjects I'll call LBUT objects. Haha, LBUTS.
 #>
 
 [cmdletbinding()]
 Param(
     [Parameter(ValueFromPipeline=$true)]
     [alias('Name','Computer','PCList','ComputerList')]
     [array]$ComputerName=$env:COMPUTERNAME,
     
     [switch]$WMI,
     
     [switch]$Remote,
 
     [switch]$Progress
 )
 
 Begin{
     $LastButs = @()
     $total = $ComputerName.Count
     $index = 0
 }
 Process{
     foreach($computer in $ComputerName){
 
         if($computer.name -ne $NULL){
             $computer = $computer.name
         }
 
         if($Progress){
             Write-Progress -Activity "Getting Last Boot Up Time from $computer" -Status 'Precent Completed' -PercentComplete ((($index++)/$total) * 100)
         }
 
         $communicate = $true
         $lbut = New-Object psobject -ArgumentList @{'Name'=$computer;'LastBootUpTime'=$NULL;'UpTime'=$NULL}
 
         Write-Verbose "Start on $computer"
 
         if(!(Test-Connection $computer -Count 1 -Quiet)){
             Write-Warning "Cannot Communicate with $computer"
             $communicate = $false
         }
 
         if($communicate -and !$WMI){
             if($Remote){
                 Write-Verbose 'Invoking {Get-Event Log}'
                 try{
                     $Events = Invoke-Command -ComputerName $computer -ScriptBlock {Get-EventLog -LogName System | % {if($_.EventID -eq '6005'){$_}} | Select-Object -First 1} -ErrorAction Stop
                 }
                 catch{
                     Write-Warning "Unable to Invoke {Get-Event Log} on $computer"
                     $communicate = $false   
                 }
             }
             else{
                 Write-Verbose 'Get-Event Log'
                 try{
                     $Events = Get-EventLog -LogName System -ComputerName $computer -ErrorAction Stop | % {if($_.EventID -eq '6005'){$_}} | Select-Object -First 1
                 }
                 catch{
                     Write-Warning "Unable to Get-Event Log from $computer"
                     $communicate = $false
                 }
             }
             if($communicate){
                 $lbut.LastBootUpTime = $Events.TimeGenerated
             }     
         }
         elseif($communicate){
             if($Remote){
                 Write-Verbose 'Invoking {Get-WMI Query}'
                 try{
                     $OpSys = Invoke-Command -ComputerName $computer -ScriptBlock {Get-WmiObject -Class Win32_OperatingSystem} -ErrorAction Stop
                 }
                 catch{
                     Write-Warning "Unable to Invoke {Get-WMI Query} on $computer"
                     $communicate = $false
                 }
             }
             else{
                 Write-Verbose 'Get-WMI Query'
                 try{
                     $OpSys = Get-WmiObject -Class Win32_OperatingSystem -ComputerName $computer -ErrorAction Stop
                 }
                 catch{
                     Write-Warning "Unable to Get-WMI Query from $computer"
                     $communicate = $false
                 }
             }
             if($communicate){          
                 $lbut.LastBootUpTime = [Management.ManagementDateTimeConverter]::ToDateTime($OpSys.LastBootUpTime)
             }
         }
 
         if($communicate){
             Write-Verbose 'Calculating UpTime'
             $lbut.UpTime = (Get-Date)-$lbut.LastBootUpTime
         }
 
         $LastButs += @($lbut)
 
     }
 }
 End{
     Write-Output $LastButs
 }
`

