---
Author: josh feierman
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 3554
Published Date: 2012-08-01t05
Archived Date: 2012-08-04t23
---

# execute-runspacejob - 

## Description

a function to facilitate use of background runspaces in powershell.

## Comments



## Usage



## TODO



## script

`execute-runspacejob`

## Code

`#
 #
 <#
 .SYNOPSIS
    Executes a set of parameterized script blocks asynchronously using runspaces, and returns the resulting data.
 .DESCRIPTION
   Encapsulates generic logic for using Powershell background runspaces to execute parameterized script blocks in an 
   efficient, multi-threaded fashion.
   
   For detailed examples of how to use the function, see http://awanderingmind.com/tag/execute-runspacejob/.
   
 .PARAMETER ScriptBlock
    The script block to execute. Should contain one or more parameters.
 .PARAMETER ArgumentList
   A hashtable containing data about the entity to be processed. The key should be a unique string to 
   identify the entity to be processed, such as a server name. The value should be another hashtable
   of arguments to be passed into the script block.
 .PARAMETER ThrottleLimit
   The maximum number of concurrent threads to use. Defaults to 10.
 .PARAMETER RunAsync
   If specified, the function will not wait for all the background runspaces to complete. Instead, it will return
   an array of runspace objects that can be used to further process the results at a later time.
 .NOTES
   Author: Josh Feierman
   Date: 7/15/2012
   Version: 1.1
    
 #>
 
 function Execute-RunspaceJob
 {
 	[Cmdletbinding()]
 	param
 	(
 		[parameter(mandatory=$true)]
 		[System.Management.Automation.ScriptBlock]$ScriptBlock,
 		[parameter(mandatory=$true,ValueFromPipeline=$true)]
 		[System.Collections.Hashtable]$ArgumentList,
 		[parameter(mandatory=$false)]
 		[int]$ThrottleLimit = 10,
     [parameter(mandatory=$false)]
     [switch]$RunAsync
 	)
 
 	begin
   {
     try
     {
       $runspacePool = [runspacefactory]::CreateRunspacePool(1,$ThrottleLimit)
       $runspacePool.Open()
       
       $runspaces = @()
       
       $data = @()
     }
     catch
     {
       Write-Warning "Error occurred initializing function setup."
       Write-Warning $_.Exception.Message
       break
     }
   }
   process
   {
     foreach ($Argument in $ArgumentList.Keys)
     {
       try
       {
         $rowIdentifier = $Argument
         Write-Verbose "Queuing item $rowIdentifier for processing."
         
         $runspaceRow = "" | Select-Object @{n="Key";e={$rowIdentifier}},
                                           @{n="Runspace";e={}},
                                           @{n="InvokeHandle";e={}}
         $powershell = [powershell]::Create()
         $powershell.RunspacePool = $runspacePool
         $powershell.AddScript($scriptBlock).AddParameters($ArgumentList[$rowIdentifier]) | Out-Null
         
         $runspaceRow.Runspace = $powershell
         $runspaceRow.InvokeHandle = $powershell.BeginInvoke()
         $runspaces += $runspaceRow
       }
       catch
       {
         Write-Warning "Error occurred queuing item '$Argument'."
         Write-Warning $_.Exception.Message
       }
     }
   }
   
   end
   {
     try
     {
       if ($RunAsync)
       {
         Write-Output $runspaces
       }
       else
       {
         $totalCount = $runspaces.Count
     
         while (($runspaces | Where-Object {$_.InvokeHandle.IsCompleted -eq $false}).Count -gt 0)
         {
           $completedCount = ($runspaces | Where-Object {$_.InvokeHandle.IsCompleted -eq $true}).Count
           Write-Verbose "Completed $completedCount of $totalCount"
           Start-Sleep -Seconds 1
         }
         
         foreach ($runspaceRow in $runspaces)
         {
           try
           {
             Write-Verbose "Retrieving data for item $($runspaceRow.Key)."
             if ($runspaceRow.Runspace.InvocationStateInfo.State -eq "Failed")
             {
               $errorMessage = $runspaceRow.Runspace.InvocationStateInfo.Reason.Message
               Write-Warning "Processing of item $($runspaceRow.Key) failed with error: $errorMessage"
             }
             else
             {
               $data += $runspaceRow.Runspace.EndInvoke($runspaceRow.InvokeHandle)
             }
           }
           catch
           {
             Write-Warning "Error occurred processing result of runspace for set '$($runspaceRow.Key)'."
             Write-Warning $_.Exception.Message
           }
           finally
           {
             $runspaceRow.Runspace.dispose()
           }
           
         }
         
         $runspacePool.Close()
         
         Write-Output $data
       }
     }
     catch
     {
       Write-Warning "Error occurred processing returns of runspaces."
       Write-Warning $_.Exception.Message
     }
   }
 }
`

