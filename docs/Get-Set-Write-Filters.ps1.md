---
Author: adam bertram
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4720
Published Date: 2013-12-19t14
Archived Date: 2013-12-22t06
---

# get/set write filters - 

## Description

get and set-writefilter to manage embedded write filters remotely

## Comments



## Usage



## TODO



## function

`get-writefilter`

## Code

`#
 #
 Function Get-WriteFilter {
     <#
     .SYNOPSIS
     Retrieves the status of an embedded device's write filter
     .DESCRIPTION
     This function uses the sysinternals psexec exe to get the status of a write filter 
     .EXAMPLE
     Get-WriteFilter -ComputerName HOSTNAME
     .PARAMETER ComputerName
     The computer name to query. Just one.
     #>
     [CmdletBinding()]
     param (
     [Parameter(Mandatory=$True,
         ValueFromPipeline=$True,
         ValueFromPipelineByPropertyName=$True,
         HelpMessage='What computer name would you like to target?')]
     $Computername
     )
 
     begin {
         $output_props = @{'ComputerName'=$ComputerName}
     }
 
     process {
         try {
             $process_output = C:\psexec.exe -e -s -n 10 \\$ComputerName C:\Windows\System32\ewfmgr.exe c: 2> $null
             if ($process_output -match 'State.+DISABLED') {
                 $output_props.Add('State','Disabled')
             } else {
                 $output_props.Add('State','Enabled')
             }
             new-object -TypeName PSCustomObject -Property $output_props
         } catch {
             $_.Exception.Message
         }
     }
 }
 
 Function Set-WriteFilter {
     <#
     .SYNOPSIS
     Sets various properties on the write filter
     .DESCRIPTION
     This function is typically used to disable an embedded device's write filter 
     .EXAMPLE
     Set-WriteFilter -ComputerName HOSTNAME -State Disabled
     .PARAMETER ComputerName
     The computer name to query. Just one.
     .PARAMETER State
     The state in which you'd like to leave the write filter
     #>
     [CmdletBinding()]
     param (
     [Parameter(Mandatory=$True,
         ValueFromPipeline=$True,
         ValueFromPipelineByPropertyName=$True,
         HelpMessage='What computer name would you like to target?')]
     $Computername,
      [Parameter(Mandatory=$True,
         ValueFromPipeline=$False,
         ValueFromPipelineByPropertyName=$True,
         HelpMessage='What state would you like the write filter in?')]
     $State,
     [Parameter(Mandatory=$False,
         ValueFromPipeline=$False,
         ValueFromPipelineByPropertyName=$False,
         HelpMessage='What state would you like the write filter in?')]
     [switch]$Restart
     )
 
     begin {
         $output_props = @{'ComputerName'=$ComputerName}
     }
 
     process {
         try {
             $current_state = (Get-WriteFilter $Computername).State
             if (($State -eq 'Disabled') -and ($current_state -eq 'Enabled')) {
                 $process_output = C:\psexec.exe -e -s -n 10 \\$ComputerName C:\Windows\System32\ewfmgr.exe c: -disable 2> $null
                 if ($Restart.IsPresent) {
                     Restart-Computer -ComputerName $Computername -Force
                 } 
             } elseif (($State -eq 'Enabled') -and ($current_state -eq 'Disabled')) {
                  $process_output = C:\psexec.exe -e -s -n 10 \\$ComputerName C:\Windows\System32\ewfmgr.exe c: -enable 2> $null
                  if ($Restart.IsPresent) {
                     Restart-Computer -ComputerName $Computername -Force
                  }
             }
             if ($LASTEXITCODE -eq 0) {
                 $output_props.Add('Result',$true)
             } else {
                 $output_props.Add('Result',$false)
             }
             new-object -TypeName PSCustomObject -Property $output_props
         } catch {
             $_.Exception.Message
         }
     }
 }
`

