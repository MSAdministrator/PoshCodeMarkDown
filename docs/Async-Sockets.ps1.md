---
Author: oisin grehan
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 2454
Published Date: 2011-01-11t12
Archived Date: 2016-04-02t03
---

# async sockets - 

## Description

derived from oisin’s new-scriptblockcallback, this is an example of how to do asynchronous sockets in powershell.  the two functions below enable simple “expect” scripting of socket communications.

## Comments



## Usage



## TODO



## class

`open-socket`

## Code

`#
 #
 if (-not ("CallbackEventBridge" -as [type])) {
    Add-Type @"
       using System;
       
       public sealed class CallbackEventBridge
       {
           public event AsyncCallback CallbackComplete = delegate { };
 
           private CallbackEventBridge() {}
 
           private void CallbackInternal(IAsyncResult result)
           {
               CallbackComplete(result);
           }
 
           public AsyncCallback Callback
           {
               get { return new AsyncCallback(CallbackInternal); }
           }
 
           public static CallbackEventBridge Create()
           {
               return new CallbackEventBridge();
           }
       }
 "@ }
 
 
 function Open-Socket {
 #.Synopsis
 #.Parameter Server
 #.Parameter Port
 #.Parameter SocketType
 #
 #.Parameter ProtocolType
 #
 #.Example
 param(
    [Parameter(Mandatory=$true, Position=0)]
    [Alias("Host")]
 	[string]$Server
 , 
    [Parameter(Mandatory=$true, Position=1)]
    [int]$Port
 , 
    [Parameter()]
    [System.Net.Sockets.SocketType]$SocketType = "Stream"
 ,
    [Parameter()]
    [System.Net.Sockets.ProtocolType]$ProtocolType = "Tcp"
 )
 end {
 	$socket = new-object System.Net.Sockets.Socket "InterNetwork", $SocketType, $Protocol
 	$socket.Connect($Server, $Port)
    Write-Output $socket
 }
 }
 
 
 function Expect-String {
 #.Synopsis
 #.Description
 #.Parameter Socket
 #.Parameter Expect
 #.Parameter SendFirst
 #.Parameter BufferLength
 #
 #.Parameter OutputBuffer
 #.Parameter Wait
 #.Notes
 #.Example
 #
 #
 #
 #.Example
 #
 
 param(
    [Parameter(Mandatory=$true, Position=1, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
    [System.Net.Sockets.Socket]$Socket
 ,
    [Parameter(Position=0,ValueFromPipelineByPropertyName=$true)]
    [String[]]$Expect = @()
 ,
    [String]$SendFirst
 ,
    [Parameter(ValueFromPipelineByPropertyName=$true)]
    [Int]$BufferLength = 100
 ,
    [Parameter(ValueFromPipelineByPropertyName=$true)]
    [Alias("Output")]
    [System.Collections.Generic.List[String]]$OutputBuffer = $(New-Object System.Collections.Generic.List[String])
 ,
    [double]$Wait
 )
 begin {
    if($SendFirst) {
       $Socket.Send(([char[]]$SendFirst)) | Out-Null
    }
    $Expecting = $Expect -as [bool]
    
    New-Object PSObject @{
       Bridge = [CallbackEventBridge]::create()
       Buffer = New-Object System.Byte[] $BufferLength
       BufferLength = $BufferLength
       Received = New-Object System.Text.StringBuilder
       Output = $OutputBuffer
       Socket = $Socket
       Complete = $false
       Expecting = $Expect
    } | Tee-Object -Variable State
 }
 
 process {
    Trap [System.ObjectDisposedException] {
       Write-Warning "The socket was forcibly closed"
    }
    
   
    Register-ObjectEvent -input $State.Bridge -EventName CallbackComplete -action {
 		param($AsyncResult) 
       
       Trap [System.ObjectDisposedException] {
          Write-Warning "The socket was forcibly closed"
          $as.Complete = "ERROR: The socket was forcibly closed"
       }
       
       $as = $AsyncResult.AsyncState
       
       $read = $as.Socket.EndReceive( $AsyncResult )
       Write-Verbose "Reading $read"
       if($read -gt 0) {
          $as.Received.Append( ([System.Text.Encoding]::ASCII.GetString($as.Buffer, 0, $read)) ) | Out-Null
       }
       
       if(($as.Expecting.Count -gt $as.Output.Count) -and $as.Received.Length -gt $as.Expecting[$as.Output.Count].Length -and $as.Expecting[$as.Output.Count]) {
          $Expecting = $as.Expecting[$as.Output.Count]
          Write-Verbose "Expecting $Expecting"
          $StartIndex = [Math]::Max(($as.Received.Length - [Math]::Max(($as.BufferLength * 2), $Expecting.Length)), 0)
          $Length = $as.Received.Length - $StartIndex
          
          $match = [regex]::Match( $as.Received.ToString( $StartIndex, $Length ), $Expecting )
          if( $match.Success ) {
             $match | Out-String | Write-Verbose 
             $matchEnd = $StartIndex + $match.Index + $match.Length
             $as.Output += $as.Received.ToString(0, $matchEnd)
             $as.Received = New-Object System.Text.StringBuilder $as.Received.ToString($matchEnd, $as.Received.Length - $matchEnd)
             if($as.Expecting.Count -eq $as.Output.Count) {
                $as.Complete = "Success: All matches found"
             }
          }
       }
       
       if($read -eq $as.BufferLength) {
          $as.Socket.BeginReceive( $as.Buffer, 0, $as.BufferLength, "None", $as.Bridge.Callback, $as ) | Out-Null
       } else {
          if($as.Expecting.Count -eq 0 -or $as.Expecting[-1].Length -eq 0) {
             $as.Output += $as.Received.ToString()
             $as.Received = New-Object System.Text.StringBuilder
          }
          $as.Complete  = $true
       }
 	} | Out-Null
 
    Write-Verbose "Begin Receiving"
 }
 end {
    if($Wait) {
       while($Wait -gt 0 -and !$State.Complete) {
          Sleep -milli 500
          $Wait -= 0.5
          Write-Progress "Waiting to Receive" "Still Expecting Output $Expect" -SecondsRemaining $Wait
       }
    }
 }
 }
`

