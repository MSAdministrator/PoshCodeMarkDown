---
Author: halr9i000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 803
Published Date: 2009-01-14t09
Archived Date: 2016-04-19t06
---

# out-default url launcher - 

## Description

this bad boy is from an email by bruce payette—the master.  here are some notes from the email

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 #
 function Out-Default `
 {
    [CmdletBinding(ConfirmImpact="Medium")]
    param(
        [Parameter(ValueFromPipeline=$true)] `
        [System.Management.Automation.PSObject] $InputObject
    )
 
    begin
    {
      $wrappedCmdlet = $ExecutionContext.InvokeCommand.GetCmdlet(
        "Out-Default")
      $sb = { & $wrappedCmdlet @PSBoundParameters }
      $__sp = $sb.GetSteppablePipeline()
      $__sp.Begin($pscmdlet)
    }
    process {
 
        $do_process = $true
        if ($_ -is [System.Management.Automation.ErrorRecord])
        {
            if ($_.Exception -is [System.Management.Automation.CommandNotFoundException])
            {
                $__command = $_.Exception.CommandName
                if (test-path -path $__command -pathtype container)
                {
                    set-location $__command
                    $do_process = $false
                }
                elseif ($__command -match '^http://|\.(com|org|net|edu)$')
                {
                    if ($matches[0] -ne "http://") { $__command = "HTTP://" + $__command }
                    [diagnostics.process]::Start($__command)
                    $do_process = $false
                }
            }
        }
        if ($do_process)
        {
            $global:LAST = $_;
            $__sp.Process($_)
        }
    }
    end { $__sp.End() }
 }
`

