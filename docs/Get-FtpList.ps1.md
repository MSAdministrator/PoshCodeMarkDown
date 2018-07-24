---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1692
Published Date: 2010-03-11t09
Archived Date: 2017-04-11t01
---

# get-ftplist - 

## Description

a function to get a file listing via ftp and parse it into objects.

## Comments



## Usage



## TODO



## function

`get-ftplist`

## Code

`#
 #
 
 function Get-FtpList {
 #.Synopsis
 #.Description
 #
 #.Parameter Path
 #.Parameter Credentials
 #.Example
 #
 #
 #
 #.Notes
 #.Link 
 #.Link
 #
 [CmdletBinding()]
 Param(
    [Parameter(Mandatory=$true)]
    [string]$Path
 ,
    [Parameter(Mandatory=$false)]
    [System.Net.ICredentials]$Credential
 )
    
    [System.Net.FtpWebRequest]$request = [System.Net.WebRequest]::Create($path)
    $request.Method = [System.Net.WebRequestMethods+FTP]::ListDirectoryDetails
    if($credential) { $request.Credentials = $credential }
    $response = $request.GetResponse()
    
    if( $response.StatusCode -ne "CommandOK" ) {
       throw "Response Status: $($response.StatusDescription)"
    }
    
    $global:ftpOutputCache = (Receive-Stream $response.GetResponseStream())
    $list = $global:ftpOutputCache -replace "(?:.|\n)*<PRE>\s+((?:.*|\n)+)\s+</PRE>(?:.|\n)*",'$1' -split "`n"
    $( foreach($line in $list | ? { $_ }) {
       [DateTime]$date, [string]$length, [string]$url, [string]$name, $null = $line -split ' + | <A HREF="|">|</A>'
       
       New-Object PSObject -Property @{ 
          LastWriteTime = $date
          FullName      = $path.trim().trim('/') + '/' + $url
          Length        = $(if($length -eq (($length -as [int]) -as [string])) { [int]$length } else { $null })
          Type          = $(if($length -eq (($length -as [int]) -as [string])) { "File" } else { $length })
          Name          = $name
       }
    ) | Sort Type | %{ $_.PSTypeNames.Insert(0, "Huddled.FtpItemInfo"); $_ } | Write-Output  
 }
 
 
 
 function Receive-Stream {
 #.Synopsis
 #.Description
 #.Param reader
 #.Param fileName
 #.Param encoding
 param( [System.IO.Stream]$reader, $fileName, $encoding = [System.Text.Encoding]::GetEncoding( $null ) )
    
    if($fileName) {
       $writer = new-object System.IO.FileStream $fileName, "Create"
    } else {
       [string]$output = ""
    }
        
    [byte[]]$buffer = new-object byte[] 4096
    [int]$total = [int]$count = 0
    do
    {
       $count = $reader.Read($buffer, 0, $buffer.Length)
       if($fileName) {
          $writer.Write($buffer, 0, $count)
       } else {
          $output += $encoding.GetString($buffer, 0, $count)
       }
    } while ($count -gt 0)
 
    $reader.Close()
    if(!$fileName) { $output }
 }
`

