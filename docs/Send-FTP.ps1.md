---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1625
Published Date: 2011-02-03t02
Archived Date: 2014-01-03t10
---

# send-ftp - 

## Description

a little send-ftp script to support uploading files. this version is simple

## Comments



## Usage



## TODO



## function

`send-ftp`

## Code

`#
 #
    function Send-FTP {
       Param(
          $Server = "ilncenter.net"
       ,  $Credentials = $(Get-Credential)
       ,  [Parameter(ValueFromPipeline=$true)]
          $LocalFile
       ,  $Path = "/"
       ,  $RemoteFile = $(Split-Path $LocalFile -Leaf)
       ,  $ProgressActivity = "Uploading $LocalFile"
       )
       Process {
          if( -not (Test-Path $LocalFile) ) {
             Throw "File '$LocalFile' does not exist!"
          }
 
          $upload = "ftp://" + $Server + ( Join-Path (Join-Path "/" $Path) $RemoteFile ) -replace "\\", "/"
          #$Upload = $upload
          $total = (gci $LocalFile).Length
 
          Write-Debug $upload
          $request = [Net.FtpWebRequest]::Create($upload)
 
          $request.Method = [System.Net.WebRequestMethods+Ftp]::UploadFile
          $request.Credentials = $Credentials.GetNetworkCredential()
          $request.UsePassive = $true
          $request.UseBinary = $true
          $request.KeepAlive = $false
 
          try {
             $read = [IO.File]::OpenRead( (Convert-Path $LocalFile) )
             $write = $request.GetRequestStream();
             
             $buffer = new-object byte[] 20KB
             $offset = 0
             $progress = 0
 
             do {
                $offset = $read.Read($buffer, 0, $buffer.Length)
                $progress += $offset
                $write.Write($buffer, 0, $offset);
                Write-Progress $ProgressActivity "Uploading" -Percent ([int]($progress/$total * 100)) -Parent $ParentProgressId
             } while($offset -gt 0)
         
          } finally {
             Write-Debug "Closing Handles"
             $read.Close()
             $write.Close()
          }
       }
    }
`

