---
Author: karl prosser
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 796
Published Date: 2009-01-11t23
Archived Date: 2016-08-09t21
---

# powersmug - sync smugmug - 

## Description

syncronize local folders with smugmug using this powershell script.

## Comments



## Usage



## TODO



## script

`get-md5`

## Code

`#
 #
 #
 #
 
 
 $ApiKey = 'uVUvCbXP3f6MgO9wIRJn21YCIgEidVly'
 $EmailAddress = '[SmugMug Email]'
 $Password = '[SmugMug Password]'
 $PhotoDirectory = '[Path To Photos]'
 $filesToInclude = @('*.jpg','*.png','*.tif','*.cr2')
 $worldSearchable = 1
 $smugSearchable = 1
 
 $watermark=0
 $watermarkId=0
 
 $smtpServer = '[SMTP Server]'
 $smtpUser = '[SMTP User]'
 $smtpPassword = '[SMTP Password]'
 $smtpFrom = 'PowerSmug@shelltools.net'
 
 
 
 $baseUrl = 'http://api.smugmug.com/hack/rest/1.2.0/'
 $userAgent = 'PowerSmug v1.0'
 $logFile = 'PowerSmug.log'
 $script:SessionId = $null
 $script:startStringState = $false
 $script:valueState = $false
 $script:arrayState = $false 
 $script:saveArrayState = $false
 $script:datFileDirty = $false
 $script:datFile = @()
 $script:albumList = $null
 $script:imageList = $null
 
 
 
 function SendEmail([string] $to, [string] $subject, [string] $msg)
 {
 	$smtpMail = new-Object System.Net.Mail.SmtpClient $smtpServer
 	$smtpMail.DeliveryMethod = [System.Net.Mail.SmtpDeliveryMethod]'Network' 
 	$smtpMail.Credentials = new-Object System.Net.NetworkCredential $smtpUser, $smtpPassword
 
 	$smtpMail.Send($smtpFrom, $to, $subject, $msg)
 }
 
 function Get-MD5([System.IO.FileInfo] $file = $(Throw 'Usage: Get-MD5 [System.IO.FileInfo]'))
 {
 	$stream = $null;
 	$cryptoServiceProvider = [System.Security.Cryptography.MD5CryptoServiceProvider];
 	$hashAlgorithm = new-object $cryptoServiceProvider
 	$stream = $file.OpenRead();
 	$hashByteArray = $hashAlgorithm.ComputeHash($stream);
 	$stream.Close();
 
 	trap
     {
 		if ($stream -ne $null) { $stream.Close(); }
 		break;
 	}
 
 	$hashByteArray | foreach { $result += $_.ToString("X2") }
 
 	return $result
 }
 
 function SendWebRequest([string]$method, $queryParams, $ssl = $false)
 {
 	$url = $baseUrl
 	if ($ssl -eq $true) {
 		$url = $url.Replace("http", "https")
 	}
 
 	$url += "?method=$method"
 
 	foreach ($key in $queryParams.Keys) {
 		$url += "&$key=" + $queryParams[$key]
 	}
 
 	$wc = New-Object Net.WebClient
 
 	$wc.Headers.Add("user-agent", $userAgent)
 	[xml]$webResult = $wc.DownloadString($url)
 	
 	CheckResponseForError $webResult
 
 	return $webResult.rsp
 }
 
 function AddFile ([System.IO.FileInfo]$file, $smugId) { 
 
 	$a = 1 | Select-Object Name, SmugId, LastModifiedTime; 
 	$a.Name = $file.Name
 	$a.SmugId = $smugId
 	$a.LastModifiedTime = $file.LastWriteTimeUtc.ToString()
 
 	$script:datFile += $a
 	
 	AppendDataFile $datFilePath $a
 }
 
 function SaveDataFile($filePath, $clearFile=$true)
 {
 	if ($script:datFileDirty -eq $false) { $script:datFile = @(); return }
 
 	[System.IO.File]::Delete($filePath)
 	
 	$sortedFile = $script:datFile | sort-Object -property Name
 	$sortedFile | Export-Csv $filePath 
 	
 	$script:datFileDirty = $false
 
 	ls $filePath | foreach { $_.Attributes = $_.Attributes -bor [System.IO.FileAttributes]::Hidden }
 
 	if ($clearFile -eq $true) {
 		$script:datFile = @()
 	}
 }
 
 function AppendDataFile($filePath, $record)
 {
 	if (test-Path $datFilePath) {
 		$newLine = [System.String]::Format('{0},{1},"{2}"', $a.Name, $a.SmugId, $a.LastModifiedTime)
 		Add-Content $filePath $newLine
 	}
 	else
 	{
 		$script:datFileDirty = $true
 		SaveDataFile $filePath $false
 	}
 }
 
 function CheckResponseForError($xmlResponse)
 {
 	if ($xmlResponse.rsp.stat -eq 'fail')	{
 		throw $webResult.rsp.err.msg
 	}
 }
 
 
 
 function Login()
 {
 	if ($script:SessionId -ne $null) { return }
 
 	$ht = @{APIKey=$ApiKey;EmailAddress=$EmailAddress;Password=$Password}
 	$loginResult = SendWebRequest "smugmug.login.withPassword" $ht $true
 	
 	if ($loginResult.stat -eq "ok") { 
 		$script:SessionId = $loginResult.Login.Session.id
 	}
 	else {
 		Throw "Error on login: " + $loginResult.Message
 	}
 }
 
 function Logout()
 {
 	if ($script:SessionId -eq $null) { return }
 
 	$ht = @{SessionID=$script:SessionId}
 	$logoutResult = SendWebRequest "smugmug.logout" $ht
 }
 
 
 
 function GetImage($name, $album)
 {
 	Login 
 	
 	$image = $script:imageList.Images | Where-Object { $_.FileName -eq $name }
 	if ($image -ne $null) { return $image }
 
 	$ht = @{SessionID=$script:SessionId;AlbumID=$album.id;Heavy=1;AlbumKey=$album.Key}
 	[xml]$script:imageList = SendWebRequest "smugmug.images.get" $ht
 	
 	return $script:imageList.Images.Image | Where-Object { $_.FileName -eq $name }	
 }
 
 function UploadFile($file, $albumName)
 {
 	$album = GetAlbum $albumName
 
 	$url = "http://upload.smugmug.com/" + $file.Name
 
 	$wc = New-Object Net.WebClient
 
 	$hash = Get-MD5 $file
 
 	$wc.Headers.Add("user-agent", $userAgent)
 	$wc.Headers.Add("ContentMd5", $hash)
 	$wc.Headers.Add("X-Smug-FileName", $file.Name)
 	$wc.Headers.Add("X-Smug-AlbumID", $album.id)
 	$wc.Headers.Add("X-Smug-SessionID", $script:SessionId)
 	$wc.Headers.Add("X-Smug-Version", "1.2.0")
 	$wc.Headers.Add("X-Smug-ResponseType", "REST")
 	$webResult = $wc.UploadFile($url, "PUT", $file.FullName)
 	[xml]$webResult = [System.Text.Encoding]::ASCII.GetString($webResult)
 	
 	CheckResponseForError $webResult
 	
 	return $webResult.rsp.Image.id
 }
 
 function UploadExistingFile($file, $id)
 {
 	Login 
 
 	$url = "http://upload.smugmug.com/" + $file.Name
 
 	$wc = New-Object Net.WebClient
 
 	$hash = Get-MD5 $file
 
 	$wc.Headers.Add("user-agent", $userAgent)
 	$wc.Headers.Add("ContentMd5", $hash)
 	$wc.Headers.Add("X-Smug-FileName", $file.Name)
 	$wc.Headers.Add("X-Smug-ImageID", $id)
 	$wc.Headers.Add("X-Smug-SessionID", $script:SessionId)
 	$wc.Headers.Add("X-Smug-Version", "1.2.0")
 	$wc.Headers.Add("X-Smug-ResponseType", "REST")
 	$webResult = $wc.UploadFile($url, "PUT", $file.FullName)
 
 	[xml]$webResult = [System.Text.Encoding]::ASCII.GetString($webResult)
 	
 	CheckResponseForError $webResult
 	
     return $webResult.rsp
 }
 
 
 
 function GetAlbumList($forceRefresh=$false) {
 	Login
 
 	if ($forceRefresh -eq $false) {
 		if ($script:albumList -ne $null) { return }
 	}
 	
 	$ht = @{SessionID=$script:SessionId}
 	$script:albumList = SendWebRequest "smugmug.albums.get" $ht
 }
 
 function GetAlbum($name)
 {
 	GetAlbumList
 
 	$album = $script:albumList.Albums.Album | Where-Object { $_.Title -eq $name }
 	if ($album -ne $null) { return $album } 
 
     Write-Host "Creating album: $name"
 	$ht = @{SessionID=$script:SessionId;Title=$name;CategoryID=0;Public=0;
     X2Larges=0;X3Larges=0;Originals=0;Watermarking=$watermark;
     WatermarkID=$watermarkId;WorldSearchable=$worldSearchable;SmugSearchable=$smugSearchable}
     $result = SendWebRequest "smugmug.albums.create" $ht
 	
 	GetAlbumList $true
 	
 	return $result.Album
 }
 
 
 
 function ProcessFile([System.IO.FileInfo]$file, $albumName)
 {
 	$photoObject = $script:datFile | Where-Object { $_.Name -eq $file.Name }
 
 	if ($photoObject -ne $null) {
 		if ($photoObject.LastModifiedTime -ne $file.LastWriteTimeUtc.ToString()) {
             write-Host "Updating existing file: " $file.FullName
 			UploadExistingFile $file $photoObject.SmugId
 			$photoObject.LastModifiedTime = $file.LastWriteTimeUtc.ToString()
             
 			$script:datFileDirty = $true
 		}
 	}
 	else {
         write-Host "Uploading new file: " $file.FullName
 		$id = UploadFile $file $albumName
 		AddFile $file $id
 	}
 }
 
 
 
 Get-ChildItem -recurse $PhotoDirectory | Where-Object { $_.Attributes -band [System.IO.FileAttributes]::Directory } | foreach {
 	if ($_.FullName.EndsWith("_") -eq $false) {
 		$datFilePath = $_.FullName + "\PowerSmug.dat"
 		if (test-Path $datFilePath) {
 			[array]$script:datFile = import-Csv $datFilePath
 		}
 	
 		$albumName = $_.FullName.Remove(0, $PhotoDirectory.Length).Trim('\')
 	
 		$path = $_.FullName + "\*"
 		foreach ($file in get-ChildItem $path -include $filesToInclude) {
 			ProcessFile $file $albumName
 		}
 		SaveDataFile $datFilePath
 		$script:imageList = $null
 	}
 }
 
 Logout
 
 $date = Get-Date
 Write-Host "Script Completed: $date"
 
 trap [Exception] {
 		$date = Get-Date
 		$Msg = $date.ToString() + " ; " + $_.Exception.GetType().FullName + " ; " + $_.Exception.Message
 		Add-Content $logFile $Msg
 		SendEmail $EmailAddress "PowerSmug Error" $Msg
 		break
 	}
 
`

