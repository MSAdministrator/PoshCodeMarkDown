---
Author: thedude
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6354
Published Date: 2016-05-23t23
Archived Date: 2016-05-28t20
---

# start-namedpipe - 

## Description

start-namedpipe

## Comments



## Usage



## TODO



## function

`start-namedpipe`

## Code

`#
 #
 
 function Start-NamedPipe {
     [OutputType([System.Management.Automation.Job])]
     [CmdletBinding()]
     param(
         [Parameter(Mandatory)]
         [string] $Name
     )
 
     $job = Start-Job -Name "Named Pipe: $Name" -ScriptBlock {
 
         $pipe = New-Object System.IO.Pipes.NamedPipeServerStream($using:Name)
         try {
             $pipe.WaitForConnection()
 
             $sr = New-Object System.IO.StreamReader($pipe)
             try {
                 while ($null -ne ($data = $sr.ReadLine())) {
                     $data
                 }
             }
             finally { 
                 $sr.Dispose()
             }
         }
         finally {
             $pipe.Dispose()
         }
 
     }
 
     $job
 }
 
 function Write-NamedPipe {
     [CmdletBinding()]
     param(
         [Parameter(Mandatory)]
         [string] $Name
     )
 
     $pipe = New-Object System.IO.Pipes.NamedPipeClientStream($Name);
     try {
         $pipe.Connect();
 
         $sw = New-Object System.IO.StreamWriter($pipe);
         try {
             $sw.WriteLine("Go"); 
             $sw.WriteLine("start abc 123"); 
             $sw.WriteLine('exit'); 
         }
         finally {
             $sw.Dispose();
         }
     }
     finally {
         $pipe.Dispose();
     }
 }
 
 function Read-NamedPipe {
     [CmdletBinding()]
     param(
         [Parameter(Mandatory)]
         [string] $Name
     )
 
     $job = Get-Job -Name "Named Pipe: $Name"
     $job | Receive-Job
 }
`

