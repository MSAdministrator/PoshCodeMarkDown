---
Author: parul jain
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4825
Published Date: 2015-01-22t01
Archived Date: 2015-08-28t03
---

# hadoop hdfs client - 

## Description

put and get large files to and from a hadoop cluster

## Comments



## Usage



## TODO



## script

`hdfs-put`

## Code

`#
 #
 <#
 .SYNOPSIS
 	Hadoop HDFS client for Windows
 
 .DESCRIPTION
 	I have data files on my Windows machine I want to send to a Hadoop cluster for map-reduce. I also want to get back the reduced data back 		 to my Windows system, perhaps to analyze with Excel or QlikView. And I want to be able to do this without installing Hadoop clients on my Windows machine. One way to do that is to SFTP or SCP files to/from one of the Hadoop cluster nodes, and then use hadoop fs commands to put/get from HDFS. However this needs twice the storage space on the cluster. A better way is needed.
 
 Hadoop natively offers a REST API to HDFS called WebHDFS. It is a pretty straightforward API that can be used with cURL. However Windows does not natively come with cURL and also it is not practical to transfer large multi-GB files with cURL. So I decided to whip up a PowerShell script that can put, get, rename, delete files, and also make and delete directories. The script will only work on a cluster where security is not enabled.
 
 To use the REST API you will need to enable WebHDFS on all nodes. To do so edit the hdfs-site.xml on each node to add the following:
 
         <property>
            <name>dfs.webhdfs.enabled</name>
            <value>true</value>
         </property>
 
 Also because the REST API will re-direct the request to the node that has the data, or where Hadoop determines the data should be stored in case of a PUT request, your Windows machine should be able to resolve names of all nodes in the cluster to IP addresses either through DNS or local hosts file.
 
 .NOTES
 	Author         : Parul Jain paruljain@hotmail.com
 	Prerequisite   : PowerShell V3 or higher
 
 .LINK
 	http://usehadoop.blogspot.com/2013/11/interacting-with-hdfs-from-windows.html
 
 .EXAMPLE
 	Hdfs-List -hostname nameNode -hdfsPath /user/jack
 .EXAMPLE
 	Hdfs-Mkdir -hostname nameNode -hdfsPath /user/jack/folder1 -user jack
 .EXAMPLE
 	Hdfs-PutFile -hostname nameNode -hdfsPath /user/jack/folder1/myfile.txt -localPath c:\myfile.txt -user jack
 .EXAMPLE
 	Hdfs-PutFile -hostname nameNode -hdfsPath /user/jack/folder1/myfile.txt -localPath c:\myfile2.txt -user jack -mode append
 .EXAMPLE
 	Hdfs-PutFile -hostname nameNode -hdfsPath /user/jack/folder1/myfile.txt -localPath c:\myfile3.txt -user jack -mode overwrite
 .EXAMPLE
 	Hdfs-Rename -hostname nameNode -hdfsPath /user/jack/folder1/myfile.txt -hdfsNewPath /user/jack/folder1/newfile.txt -user jack
 .EXAMPLE
 	Hdfs-GetFile -hostname nameNode -hdfsPath /user/jack/folder1/newfile.txt -localPath c:\myfile4.txt -user jack
 .EXAMPLE
 	Hdfs-GetFile -hostname nameNode -hdfsPath /user/jack/folder1/newfile.txt -localPath c:\myfile4.txt -user jack -length 2000
 .EXAMPLE
 	Hdfs-GetFile -hostname nameNode -hdfsPath /user/jack/folder1/newfile.txt -localPath c:\myfile4.txt -user jack -overwrite
 .EXAMPLE
 	Hdfs-GetFile -hostname nameNode -hdfsPath /user/jack/folder1/newfile.txt -localPath c:\myfile4.txt -user jack -append
 .EXAMPLE
 	Hdfs-Remove -hostname nameNode -hdfsPath /user/jack -user jack -recurse
 #>
 
 function Hdfs-Put {
     param (
         [Parameter(Mandatory=$True)][string]$hostname,
         [Parameter(Mandatory=$False)][int]$port = 50070,
         [Parameter(Mandatory=$True)][byte[]]$data,
         [Parameter(Mandatory=$True)][string]$hdfsPath,
         [Parameter(Mandatory=$True)][string]$user,
         [Parameter(Mandatory=$False)][ValidateSet('open', 'append', 'overwrite')][string]$mode = 'open'
     )
          
     if (!(Test-Path $localPath)) { throw "$localPath does not exist" }
     if ($hdfsPath -notmatch '^/') { throw "hdfsPath must start with a /" }
     $method = 'PUT'
     $uri = "http://${hostname}:$port/webhdfs/v1${hdfspath}?op=CREATE&overwrite=false&user.name=$user"
     if ($mode -match 'append') { $uri = "http://${hostname}:$port/webhdfs/v1${hdfspath}?op=APPEND&user.name=$user"; $method = 'POST' }
     if ($mode -match 'overwrite') { $uri = "http://${hostname}:$port/webhdfs/v1${hdfspath}?op=CREATE&overwrite=true&user.name=$user" }
     $wr = [System.Net.WebRequest]::Create($uri)
     $wr.Method = $method
     $wr.AllowAutoRedirect = $false
     $response = $wr.GetResponse()
     if ($response.StatusCode -ne 'TemporaryRedirect') {
         throw 'Error: Expected temporary redirect, got ' + $response.StatusCode
     }
     $wr = [System.Net.WebRequest]::Create($response.Headers['Location'])
     $wr.Method = $method
     $wr.ContentLength = $data.Length
     $requestBody = $wr.GetRequestStream()
     $requestBody.Write($data, 0, $data.Length)
     $requestBody.Close()
  
     $wr.GetResponse()
 }
  
 function Hdfs-Get {
     param (
         [Parameter(Mandatory=$True)][string]$hostname,
         [Parameter(Mandatory=$False)][int]$port = 50070,
         [Parameter(Mandatory=$True)][string]$hdfsPath,
         [Parameter(Mandatory=$False)][string]$user,
         [Parameter(Mandatory=$False)][long]$offset = 0,
         [Parameter(Mandatory=$False)][long]$length = 67108864
     )
          
     $uri = "http://${hostname}:$port/webhdfs/v1${hdfspath}?op=OPEN&offset=$offset&length=$length"
    
     if ($user) { $uri += '&user.name=' + $user }
     $wr = [System.Net.WebRequest]::Create($uri)
     $response = $wr.GetResponse()
     $responseStream = $response.GetResponseStream()
     $br = New-Object System.IO.BinaryReader($responseStream)
     $br.ReadBytes($response.ContentLength)
     $br.Close()
     $responseStream.Close()
 }
  
 function Hdfs-List {
     param (
         [Parameter(Mandatory=$True)][string]$hostname,
         [Parameter(Mandatory=$False)][int]$port = 50070,
         [Parameter(Mandatory=$True)][string]$hdfsPath
     )
     if ($hdfsPath -notmatch '^/') { throw "hdfsPath must start with a /" }
     $fileStatus= Invoke-RestMethod -Method Get -Uri "http://${hostname}:$port/webhdfs/v1${hdfsPath}?op=LISTSTATUS"
     foreach ($item in $fileStatus.FileStatuses.FileStatus) {
         $item.accessTime = Convert-FromEpochTime $item.accessTime
         $item.modificationTime = Convert-FromEpochTime $item.modificationTime
         $item
     }
 }
  
 function Hdfs-Remove {
     param (
         [Parameter(Mandatory=$True)][string]$hostname,
         [Parameter(Mandatory=$False)][int]$port = 50070,
         [Parameter(Mandatory=$True)][string]$hdfsPath,
         [Parameter(Mandatory=$True)][string]$user,
         [switch]$recurse
     )
     if ($hdfsPath -notmatch '^/') { throw "hdfsPath must start with a /" }
     if ($recurse) { $rec = 'true' } else { $rec = 'false' }
     $result = Invoke-RestMethod -Method Delete -Uri "http://${hostname}:$port/webhdfs/v1${hdfsPath}?op=DELETE&recursive=$rec&user.name=$user"
     $result.boolean
 }
  
 function Hdfs-Mkdir {
     param (
         [Parameter(Mandatory=$True)][string]$hostname,
         [Parameter(Mandatory=$False)][int]$port = 50070,
         [Parameter(Mandatory=$True)][string]$hdfsPath,
         [Parameter(Mandatory=$True)][string]$user,
         [Parameter(Mandatory=$False)][string]$permission
     )
     if ($hdfsPath -notmatch '^/') { throw "hdfsPath must start with a /" }
     if ($permission) {
         $result = Invoke-RestMethod -Method Put -Uri "http://${hostname}:$port/webhdfs/v1${hdfsPath}?op=MKDIRS&permission=$permission&user.name=$user" }
     else { $result = Invoke-RestMethod -Method Put -Uri "http://${hostname}:$port/webhdfs/v1${hdfsPath}?op=MKDIRS&user.name=$user" }
     $result.boolean
 }
  
 function Hdfs-Rename {
     param (
         [Parameter(Mandatory=$True)][string]$hostname,
         [Parameter(Mandatory=$False)][int]$port = 50070,
         [Parameter(Mandatory=$True)][string]$hdfsPath,
         [Parameter(Mandatory=$True)][string]$hdfsNewPath,
         [Parameter(Mandatory=$True)][string]$user
     )
     if ($hdfsPath -notmatch '^/') { throw "hdfsPath must start with a /" }
     if ($hdfsNewPath -notmatch '^/') { throw "hdfsNewPath must start with a /" }
     $result = Invoke-RestMethod -Method Put -Uri "http://${hostname}:$port/webhdfs/v1${hdfsPath}?op=RENAME&user.name=$user&destination=$hdfsNewPath"
     $result.boolean
 }
  
 function Convert-FromEpochTime ([long]$epochTime) {
     [TimeZone]::CurrentTimeZone.ToLocalTime(([datetime]'1/1/1970').AddSeconds($epochTime/1000))
 }
  
 function Hdfs-PutFile {
    param (
         [Parameter(Mandatory=$True)][string]$hostname,
         [Parameter(Mandatory=$False)][int]$port = 50070,
         [Parameter(Mandatory=$True)][string]$localPath,
         [Parameter(Mandatory=$True)][string]$hdfsPath,
         [Parameter(Mandatory=$True)][string]$user,
         [Parameter(Mandatory=$False)][int]$sliceSize = 67108864,
         [Parameter(Mandatory=$False)][ValidateSet('open', 'append', 'overwrite')][string]$mode = 'open'
     )
        
     try {
         $br = New-Object System.IO.BinaryReader([System.IO.File]::Open($localPath, [System.IO.FileMode]::Open))
     } catch { throw $error[0].Exception.Message }
     $total = $br.BaseStream.Length
     $sent = 0
     $firstRun = $true
    
     do {
         Write-Progress -Activity "Copying $localPath to HDFS on $hostname" -PercentComplete ($sent/$total * 100)
         $data = $br.ReadBytes($sliceSize)
         try {
             Hdfs-Put -hostname $hostname -port $port -user $user -hdfsPath $hdfsPath -data $data -mode $mode | out-null
         } catch { $br.Close(); throw $error[0].Exception.Message }
         $sent += $sliceSize
         if ($firstRun) { $firstRun = $false; $mode = 'append' }
     } while ($data.LongLength -eq $sliceSize)
     $br.Close()
 }
  
 function Hdfs-GetFile {
     param (
         [Parameter(Mandatory=$True)][string]$hostname,
         [Parameter(Mandatory=$False)][int]$port = 50070,
         [Parameter(Mandatory=$True)][string]$hdfsPath,
         [Parameter(Mandatory=$False)][string]$user,
         [Parameter(Mandatory=$False)][string]$localPath,
         [Parameter(Mandatory=$False)][long]$length,
         [switch]$append,
         [switch]$overwrite
     )
     if ($append -and $overwrite) { throw 'Cannot use -append and -overwrite together' }
     $mode = [System.IO.FileMode]::CreateNew
     if ($append) {$mode = [System.IO.FileMode]::Append}
     if ($overwrite) {$mode = [System.IO.FileMode]::Create}
    
     try {
         $bw = New-Object System.IO.BinaryWriter([System.IO.File]::Open($localPath, $mode))
     } catch { throw $error[0].Exception.Message }
    
     $fileAttribs = Hdfs-List -hostname $hostname -hdfsPath $hdfsPath -port $port
     if (!$length) { $length = $fileAttribs.length }
     $blockSize = $fileAttribs.blockSize
     if ($length -lt $blockSize) { $blockSize = $length }
     $got = 0
    
     do {
         Write-Progress -Activity "Copying $hdfsPath to $localPath" -PercentComplete ($got/$length * 100)
        
         try {
             [byte[]]$data = Hdfs-Get -hostname $hostname -port $port -user $user -hdfsPath $hdfsPath -offset $got -length $blockSize
         } catch { $bw.Close(); throw $error[0].Exception.Message }
         try {
             $bw.Write($data)
         } catch { $bw.Close(); throw $error[0].Exception.Message }
         $got += $data.LongLength
     } while ($got -lt $length)
     $bw.Close()
 }
`

