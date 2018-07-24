---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6644
Published Date: 2016-12-07t18
Archived Date: 2016-12-09t05
---

# webserver - 

## Description

just an idea for how to handle web requests in powershell

## Comments



## Usage



## TODO



## function

`start-webserver`

## Code

`#
 #
 
 
 function Start-WebServer {
    param($pattern = "test")
 
    $script:listener = New-Object System.Net.HttpListener
    $script:listener.Prefixes.Add("http://+:80/${pattern}/")
    $script:listener.Start()
 }
 
 function Stop-WebServer {
    $script:listener.Stop()
 }
 
 function Wait-Request {
    [CmdletBinding()]param()
    $Context = $script:listener.GetContext()
    Write-Verbose ($Context.Request.HttpMethod + " " + $Context.Request.Url)
    switch($Context.Request.HttpMethod) {
       "GET" {
          $Context | Send-Response "POST HANDLER"
       }
       "POST" {
          $Context | Receive-Request
          $Context | Send-Response ([PSCustomObject]@{Success=$True})
       }
       default {
          $Context.Response.OutputStream.Close()
       }
    }
 }
 
 function Send-Response {
    param(
       [Parameter(Mandatory,ValueFromPipelineByPropertyName)]
       $Response, 
 
       [Parameter(Position=0)]
       $Content="HELLO"
    )
    $Content = ConvertTo-Json $Content
    $buffer = [Text.Encoding]::UTF8.GetBytes($Content)
    $Response.ContentLength64 = $buffer.Length;
    $Response.OutputStream.Write( $buffer, 0, $buffer.Length )
    $Response.OutputStream.Close()
 }
 
 function Receive-Request {
    param(
       [Parameter(Mandatory,ValueFromPipelineByPropertyName)]
       $Request
    )
    $output = ""
 
    #$size = $script:DefaultBufferSize
       $size = $Request.ContentLength64 + 1
    #}
 
    Write-Verbose "Receiving up to $size"
    $buffer = New-Object byte[] $size
    do {
       $count = $Request.InputStream.Read($buffer, 0, $size)
       Write-Verbose "Received $count"
       $output += $Request.ContentEncoding.GetString($buffer, 0, $count)
    } until($count -lt $size)
 
    $Request.InputStream.Close()
    ConvertFrom-Json $output
 }
`

