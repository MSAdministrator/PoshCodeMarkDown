---
Author: noldorin
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2316
Published Date: 2011-10-21t18
Archived Date: 2017-04-08t00
---

# ftp module - 

## Description

module for communicating with ftp server

## Comments



## Usage



## TODO



## function

`get-scriptpath`

## Code

`#
 #
 <#
 .Synopsis
     Utilities for communicating with FTP server.
 #>
 
 function Get-ScriptPath {
     Split-Path $myInvocation.ScriptName 
 }
 
 Import-Module (Join-Path (Get-ScriptPath) common.psm1) -Force
 
 <#
 .Synopsis
     Lists contents of directory via FTP.
 .Parameter oath
     URL of directory to list.
 .Parameter credentials
     Log-in credentials.
 #>
 function Get-FtpList {
     [CmdletBinding()]
     param(
         [Parameter(Mandatory = $true)]
         [string] $path,
         [Parameter(Mandatory = $false)]
         [Net.ICredentials] $credentials
     )
     
     $request = [Net.WebRequest]::Create($path)
     $request.Method = [Net.WebRequestMethods+Ftp]::ListDirectory
     if ($credentials) { $request.Credentials = $credentials }
     try {
         $response = $request.GetResponse()
         return Get-Reponse($response);
     } catch [Net.WebException] {
         throw
     }
 }
 
 <#
 .Synopsis
     Downloads file via FTP.
 .Parameter path
     URL of fiel to download.
 .Parameter credentials
     Log-in credentials.
 #>
 function Get-FtpFile {
     [CmdletBinding()]
     param(
         [Parameter(Mandatory = $true)]
         [string] $path,
         [Parameter(Mandatory = $false)]
         [Net.ICredentials] $credentials
     )
     
     $request = [Net.WebRequest]::Create($path)
     $request.Method = [Net.WebRequestMethods+Ftp]::DownloadFile
     if ($credentials) { $request.Credentials = $credentials }
     try {
         $response = $request.GetResponse()
         return Get-Reponse($response);
     } catch [Net.WebException] {
         throw
     }
 }
 
 <#
 .Synopsis
     Uploads binary file via FTP.
 .Parameter path
     URL of file to upload.
 .Parameter credentials
     Log-in credentials.
 #>
 function Set-FtpFile {
     [CmdletBinding()]
     param(
         [Parameter(Mandatory = $true)]
         [string] $path,
         [Parameter(Mandatory = $true)]
         [string] $fileName,
         [Parameter(Mandatory = $false)]
         [Net.ICredentials] $credentials
     )
     
     $request = [Net.WebRequest]::Create($path)
     $request.Method = [Net.WebRequestMethods+Ftp]::UploadFile
     $request.UseBinary = $true
     if ($credentials) { $request.Credentials = $credentials }
     Set-Request $request $fileName
     try {
         $response = $request.GetResponse()
         return Get-Reponse($response);
     } catch [Net.WebException] {
         throw
     }
 }
 
 <#
 .Synopsis
     Uploads text file via FTP.
 .Parameter path
     URL of file to upload.
 .Parameter credentials
     Log-in credentials.
 #>
 function Set-FtpTextFile {
     [CmdletBinding()]
     param(
         [Parameter(Mandatory = $true)]
         [string] $path,
         [Parameter(Mandatory = $true, ValueFromPipeline = $true)]
         [String[]] $content,
         [Parameter(Mandatory = $false)]
         [Net.ICredentials] $credentials
     )
         
     begin {
         $request = [Net.WebRequest]::Create($path)
         $request.Method = [Net.WebRequestMethods+Ftp]::UploadFile
         $request.UseBinary = $false
         if ($credentials) { $request.Credentials = $credentials }
         $writer = New-Object IO.StreamWriter $request.GetRequestStream()
     } process {
         $writer.WriteLine($_)
     } end {
         $writer.Dispose()
         try {
             $response = $request.GetResponse()
             return Get-Reponse($response);
         } catch [Net.WebException] {
             throw
         }
     }
 }
 
 <#
 .Synopsis
     Creates new directory via FTP.
 .Parameter path
     URL of directory to create.
 .Parameter credentials
     Log-in credentials.
 #>
 function New-FtpDir {
     [CmdletBinding()]
     param(
         [Parameter(Mandatory = $true)]
         [string] $path,
         [Parameter(Mandatory = $false)]
         [Net.ICredentials] $credentials
     )
         
     $request = [Net.WebRequest]::Create($path)
     $request.Method = [Net.WebRequestMethods+Ftp]::MakeDirectory
     if ($credentials) { $request.Credentials = $credentials }
     try {
         $response = $request.GetResponse()
         return Get-Reponse($response);
     } catch [Net.WebException] {
         throw
     }
 }
 
 <#
 .Synopsis
     Removes directory via FTP.
 .Parameter path
     URL of directory to remove.
 .Parameter credentials
     Log-in credentials.
 #>
 function Remove-FtpDir {
     [CmdletBinding()]
     param(
         [Parameter(Mandatory = $true)]
         [string] $path,
         [Parameter(Mandatory = $false)]
         [Net.ICredentials] $credentials
     )
         
     $request = [Net.WebRequest]::Create($path)
     $request.Method = [Net.WebRequestMethods+Ftp]::RemoveDirectory
     if ($credentials) { $request.Credentials = $credentials }
     try {
         $response = $request.GetResponse()
         return Get-Reponse($response);
     } catch [Net.WebException] {
         throw
     }
 }
 
 function Set-Request([Net.WebRequest] $request, $contentFileName) {
     try {
         $readStream = [IO.File]::OpenRead( (Convert-Path $contentFileName) )
         $writeStream = $request.GetRequestStream();
         
         $buffer = New-Object byte[] 32KB
         $offset = 0
         $total = $readStream.Length
         $progress = 0
         
         do {
             $offset = $readStream.Read($buffer, 0, $buffer.Length)
             $progress += $offset
             $writeStream.Write($buffer, 0, $offset);
             Write-Progress "Uploading $contentFileName" "Uploading" `
                 -Percent ([int]($progress / $total * 100))
         } while ($offset -gt 0)
     } finally {
         $readStream.Close()
         $writeStream.Close()
     }
 }
 
 function Get-Reponse([Net.WebResponse] $response) {
     use ($reader = New-Object IO.StreamReader $response.GetResponseStream()) {
         while (($line = $reader.ReadLine()) -ne $null) {
             Write-Output $line
         }
     }
 }
`

