---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 3.5
Encoding: ascii
License: cc0
PoshCode ID: 898
Published Date: 
Archived Date: 2009-03-03t09
---

# new-namedpipe - 

## Description

the below will create a bi-directional named pipe with the name you specify in the $pipename variable.  note that .net 3.5 is required for the system.io.pipes namespace.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 [reflection.Assembly]::LoadWithPartialName("system.core") | Out-Null
 $pipeName = "pipename"
 $pipeDir = [System.IO.Pipes.PipeDirection]::InOut
 $pipe = New-Object system.IO.Pipes.NamedPipeServerStream( $pipeName, $pipeDir )
`

