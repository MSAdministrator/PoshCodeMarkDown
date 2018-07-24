---
Author: kenneth w hightower jr
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5759
Published Date: 2017-02-25t18
Archived Date: 2017-01-08t08
---

# start-scriptthreading - 

## Description

ever wanted to start some threads on a block of code but hated the headache of creating, watching, and collecting job data? this is my attempt to put a bandaid on that issue.

## Comments



## Usage



## TODO



## function

`start-scriptthreading`

## Code

`#
 #
 function Start-ScriptThreading{
 .SYNOPSIS
    
     Runs a scriptblock against multiple computers simultaneously 
 
 .DESCRIPTION
  
     The ThreadedFunction function takes an array (usually a list of computers) and a scriptblock. The scriptblock will be run against
     each item in the array expecting each item of the array to also be a parameter expected in the scriptblock. 
 
 .PARAMETER arComputers
  
     List of computers to run scriptblock against. 
 
 .PARAMETER ScriptBlock
  
     Block of script to run against list from parameter -Computers   
 
 .PARAMETER maxThreads
  
     Maximum number of threads to run at a time  
 
 .PARAMETER SleepTimer
  
     Time to wait between loops. This is a spacing timer to prevent the main loop from hogging cpu time. 
     A smaller number may create and check for jobs faster.
     A large number will allow the separate threads more cpu time.  
     Any longer than 1000 milliseconds may render the progress indicator incorectly.
     (default is 500 milliseconds)   
 
 .PARAMETER MaxWaitAtEnd
 
     Sets a timeout for threads before killing jobs. This timer is started after the last thread is created.
     (default is 180 seconds)              
 
 .NOTES
    
     Name: Start-ScriptThreading
     Author: Kenneth W Hightower JR (The13thDoctor)
     DateCreated: 11Feb2015
     DateModified: 25Feb2015
 
 
     
     To load this function into the current shell for use, or to use this file seperate from your main script,
  
     dot-source as follows '. .\ThreadedFunction.ps1'    
 
 .EXAMPLE
    
     Start-ScriptThreading -Computers 'server01', 'server02' -ScriptBlock {ping $Computer}
 
     The command 'ping' is threaded and ran against both server01 and server02. Declaring $Computer is required as is.
 
 .EXAMPLE
    
     'server01', 'server02' | Start-ScriptThreading -ScriptBlock {Get-WMIObject win32_computersystem -ComputerName $Computer}
  
     Pipes the array as Computers and gets the Win32_ComputerSystem WMI object from server01 and server02 simultaneously.   
 
 .INPUTS
 
     System.String. Function will accept an array of strings and assume each as a parameter for the provided scriptblock.
 
 .OUTPUTS
 
     System.Object. All returned data from each iteration that the scriptblock may create is returned as a single array.
            
 
 #>  
     param(
         [parameter(Mandatory=$true,ValueFromPipeline=$true)]
         [string[]]$arComputers,
         [parameter(Mandatory=$true,ValueFromPipeline=$false)]
         [scriptblock]$ScriptBlock,
         [parameter(Mandatory=$false)]
         [int]$maxThreads = 30,
         [parameter(Mandatory=$false)]
         [int]$SleepTimer = 500,
         [parameter(Mandatory=$false)]
         [int]$MaxWaitAtEnd = 180
     )
     BEGIN{
         Write-Verbose "Initializing..."
         $return = @()
         $StartTime= Get-Date
     
         $Script = [ScriptBlock]::Create( @"
 param(`$Computer)
 &{ $ScriptBlock } @PSBoundParameters
 "@ )
     $Computers = @()
     }
     PROCESS{
     
         foreach($Computer in $arComputers){
         }
     } 
     END{
         Write-Verbose "creating threads..."
         Foreach($Computer in $Computers){
         
             while ($(Get-Job -State Running | measure-object).count -ge $maxThreads){
     		    write-host "$((Get-Job -State Running | measure-object).count) threads running, please wait..."
     		    start-sleep -s 3
         
             Write-Verbose "starting $($Computer)"
             Start-Job -name $Computer -scriptblock $Script -ArgumentList $Computer
         
             $ComputersStillRunning = ""
             ForEach ($System  in $(Get-Job -state running)){
                 $ComputersStillRunning += ", $($System.name)"
             }
             $ComputersStillRunning = $ComputersStillRunning.TrimStart(", ")
             Write-Progress  -Activity "Creating Jobs" `
                             -Status "$($(Get-Job -State Running).count) threads running..." `
                             -CurrentOperation "$ComputersStillRunning" `
                             -PercentComplete ($(Get-Job -State Completed).count / $(Get-Job).count * 100)
 
             ForEach($Job in $(Get-Job -State Completed)){
                 If($Job.HasMoreData -eq "True"){
                     Write-Verbose "Job $($Job.Id) finished early"
                     $Data = Receive-Job $Job
                     $Return+=$Data
     
         Write-Verbose "Waiting for final threads..."
         $Complete = Get-date
 
         While ($(Get-Job -State Running | measure-object).count -gt 0){
         
             $ComputersStillRunning = ""
             $TimeDiff = New-TimeSpan $StartTime $(Get-Date)
             [string]$strTimingStatus = "{0} minutes, {1} seconds so far" -f $TimeDiff.Minutes, $TimeDiff.Seconds
             ForEach ($System  in $(Get-Job -state running)){
                 $ComputersStillRunning += ", $($System.name)"
             }
             $ComputersStillRunning = $ComputersStillRunning.TrimStart(", ")
             Write-Progress  -Activity "Waiting for completion..." `
                             -Status "$($(Get-Job -State Running).count) threads remaining: $strTimingStatus" `
                             -CurrentOperation "$ComputersStillRunning" `
                             -PercentComplete ($(Get-Job -State Completed).count / $(Get-Job).count * 100)
 
             ForEach($Job in $(Get-Job -State Completed)){
                 If($Job.HasMoreData -eq "True"){
                     Write-Verbose "Job $($Job.Id) just completed"
                     $Data = Receive-Job $Job
                     $return+=$Data
 
             If ($(New-TimeSpan $Complete $(Get-Date)).totalseconds -ge $MaxWaitAtEnd){
         	    Write-Verbose "Killing all jobs still running, tired of waiting..."
         	    Get-Job -State Running | Remove-Job -Force
     	    }
 
             Start-Sleep -Milliseconds $SleepTimer
         }
     
         ForEach($Job in $(Get-Job -State Completed)){
             If($Job.HasMoreData -eq "True"){
                 Write-Verbose "Job $($Job.Id) completed late"
                 $Data = Receive-Job $Job
                 $return+=$Data
 
         Write-Verbose "Cleaning up threads"
         Get-Job | Remove-Job -Force | Out-Null
 
         return $return
     }
`

