---
Author: test-tcpport
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6329
Published Date: 2016-04-29t11
Archived Date: 2016-06-03t00
---

# test-tcpport - 

## Description

test-tcpport

## Comments



## Usage



## TODO



## function

`test-tcpport`

## Code

`#
 #
 function Test-TCPPort
 {
     [CmdletBinding()]
     param
     (
         [Parameter(Position=0,
                    ValueFromPipeline=$true,
                    ValueFromPipelineByPropertyName=$true)]
         [Alias('hostname')]
         [Alias('cn')]
         [string[]]$Name = $env:COMPUTERNAME,
 
         [Parameter(Position=2)]
         [int]$Port=80,
 
         [Parameter(Position=3)]
         [Int]$TimeOut = 1000
     )
 
     BEGIN { 
 	    $Socket = New-Object System.Net.Sockets.TCPClient
     }
 
     PROCESS
     {
         $Connect = $Socket.BeginConnect($Name,$Port,$null,$null)
 	    if ( $Connect.IsCompleted )
 	    {
 	    	$Wait = $Connect.AsyncWaitHandle.WaitOne($TimeOut,$false)
 	    	if(!$Wait) 
 	    	{
 	    		return $false 
 	    	} 
 	    	else
 	    	{
 	    		$Socket.EndConnect($Connect)
 	    		return $true
 	    	}
 	    }
 	    else
 	    {
 	    	return $false
 	    }
         }
 
     END { 
         $Socket.Close()
     }
 }
`

