---
Author: justin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4543
Published Date: 2014-10-22t17
Archived Date: 2017-05-22t02
---

# invoke-async - 

## Description

quick way to do processes a ton of jobs, for example ping 100 boxes at once

## Comments



## Usage



## TODO



## function

`invoke-async`

## Code

`#
 #
 param(
 [parameter(Mandatory=$true,ValueFromPipeLine=$true)]
 [object[]]$Set,
 [parameter(Mandatory=$true)]
 [string] $SetParam,
 [parameter(Mandatory=$true, ParameterSetName='cmdlet')]
 [string]$Cmdlet,
 [parameter(Mandatory=$true, ParameterSetName='ScriptBlock')]
 [scriptblock]$ScriptBlock,
 Function Invoke-Async{
 [hashtable]$Params,
 [int]$ThreadCount=10,
 [switch]$Measure
 )
 Begin
 {
 
     $Threads = @()
     $Length = $JobsLeft = $Set.Length
 
     $Count = 0
     if($Length -lt $ThreadCount){$ThreadCount=$Length}
     $timer = @(1..$ThreadCount  | ForEach-Object{$null})
     $Jobs = @(1..$ThreadCount  | ForEach-Object{$null})
     
     If($PSCmdlet.ParameterSetName -eq 'cmdlet')
     {
         $CmdType = (Get-Command $Cmdlet).CommandType
         if($CmdType -eq 'Alias')
         {
             $CmdType = (Get-Command (Get-Command $Cmdlet).ResolvedCommandName).CommandType
         }
         
         If($CmdType -eq 'Function')
         {
             $ScriptBlock = (Get-Item Function:\$Cmdlet).ScriptBlock
             1..$ThreadCount | ForEach-Object{ $Threads += [powershell]::Create().AddScript($ScriptBlock)}
         }
         ElseIf($CmdType -eq "Cmdlet")
         {
             1..$ThreadCount  | ForEach-Object{ $Threads += [powershell]::Create().AddCommand($Cmdlet)}
         }
     }
     Else
     {
         1..$ThreadCount | ForEach-Object{ $Threads += [powershell]::Create().AddScript($ScriptBlock)}
     }
 
     If($Params){$Threads | ForEach-Object{$_.AddParameters($Params) | Out-Null}}
 
 }
 Process
 {
     while($JobsLeft)
     {
         for($idx = 0; $idx -lt ($ThreadCount-1) ; $idx++)
         {
             $SetParamObj = $Threads[$idx].Commands.Commands[0].Parameters| Where-Object {$_.Name -eq $SetPa
 ram}
              
             {  
                 $result = $null
                 if($threads[$idx].InvocationStateInfo.State -eq "Failed")
                 {
                     $result  = $Threads[$idx].InvocationStateInfo.Reason
                     Write-Error "Set Item: $($SetParamObj.Value) Exception: $result"
                 }
                 else
                 { 
                     $result = $Threads[$idx].EndInvoke($Jobs[$idx])
                 }
                 $ts = (New-TimeSpan -Start $timer[$idx] -End (Get-Date))
                 if($Measure)
                 {
                     new-object psobject -Property @{
                         TimeSpan = $ts
                         Output = $result
                         SetItem = $SetParamObj.Value}
                 }
                 else
                 {
                     $result
                 }
                 $Jobs[$idx] = $null
                 write-verbose "Completed: $($SetParamObj.Value) in $ts"
                 write-progress -Activity "Processing Set" -Status "$JobsLeft jobs left" -PercentComplete ((
 $length-$jobsleft)/$length*100)
             }
             {
                 write-verbose "starting: $($Set[$Count])"
                 $timer[$idx] = get-date
 uccess?
                 $Threads[$idx].AddParameter($SetParam,$Set[$Count]) | Out-Null
                 $Jobs[$idx] = $Threads[$idx].BeginInvoke()
                 $Count++
             }
         }
 
     }
 }
 End
 {
     $Threads | ForEach-Object{$_.runspace.close();$_.Dispose()}
 }
 }
`

