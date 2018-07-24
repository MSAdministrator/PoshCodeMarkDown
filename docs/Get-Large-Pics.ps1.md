---
Author: james gentile
Publisher: 
Copyright: 
Email: 
Version: 537.36
Encoding: ascii
License: cc0
PoshCode ID: 6264
Published Date: 2016-03-21t14
Archived Date: 2016-10-05t07
---

# get-large-pics.ps1 - 

## Description

get-large-pics.ps1 – scans a source directory’s pictures and stores the largest version (if larger than the original source) of each picture it can find in google image search in the target directory. usage

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 
 function global:Get-GoogleImageSearchUrl
 {
     param(
         [Parameter(Mandatory = $true)]
         [ValidateScript({ Test-Path $_ })]
         [string] $ImagePath
     )
 
     $imagepath = (get-item -ea 0 $imagepath).fullname
     $fileName = Split-Path $imagePath -Leaf
 
     $part1 = @"
 -----------------------------7dd2db3297c2202
 Content-Disposition: form-data; name="encoded_image"; filename="$fileName"
 Content-Type: image/jpeg
 
 
 "@
     $part2 = @"
 -----------------------------7dd2db3297c2202
 Content-Disposition: form-data; name="image_content"
 
 
 -----------------------------7dd2db3297c2202--
 
 "@
 
     $imageBytes = [Io.File]::ReadAllBytes($imagePath)
 
     $encoding = New-Object Text.ASCIIEncoding
     $data = $encoding.GetBytes($part1) + $imageBytes + $encoding.GetBytes($part2)
 
     $request = [Net.HttpWebRequest] ([Net.HttpWebRequest]::Create('http://images.google.com/searchbyimage/upload'))
     $request.Method = "POST"
     $request.ContentLength = $data.Length
 
     $request.AllowAutoredirect = $false
 
     $stream = $request.GetRequestStream()
     $stream.Write($data, 0, $data.Length)
     $stream.Close()        
 
     $respStream = $request.GetResponse().GetResponseStream()
 
     (New-Object Io.StreamReader $respStream).ReadToEnd() -match 'HREF\="([^"]+)"' | Out-Null
     $matches[1]
 }
 
 [Reflection.Assembly]::LoadWithPartialName("System.Web") | Out-Null
 $foundfiles = 0
 if ($args.count -lt 1)
 {
 	$sourceFolder = ".\pics2\"
 }
 else
 {
 	$sourceFolder = $args[0] 
 }
 if ($args.count -lt 2)
 {
 	$TargetFolder = join-path $sourceFolder "results"
 }
 else
 {
 	$TargetFolder = $args[1]
 }
 $sourceFolder = join-path $sourceFolder "*"
 $browserAgent = 'Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/41.0.2228.0 Safari/537.36'
 if ( (Test-Path -Path $TargetFolder) -eq $false) { md $TargetFolder }
 $files = @(dir $sourceFolder -include *.png, *.gif, *.jpg, *.jpeg|?{$_.psiscontainer -eq $false})
 for ($t=0;$t -lt $files.Count;$t++)
 {
 	$f = $files[$t]
 	"Processing file $($f.basename)$($f.extension) - #$($t) of $($files.Count)"
 	
 	$url = get-GoogleImageSearchUrl $f.fullname
     
     	$page = Invoke-WebRequest -Uri $url -UserAgent $browserAgent -TimeoutSec 30
     	$link = @($page.Links|?{$_.outertext -eq "Large"})
     	if ($link.Count -eq 0 -or $link.href -notlike "/search*") 
     	{
         	"No Large pictures found on google for $($F.basename)"
         	continue
     	}
     	$url = "www.google.com"+$link[0].href.tostring().replace("&amp;","&")
  
 	$page = Invoke-WebRequest -Uri $url -UserAgent $browserAgent -TimeoutSec 30
 	$newURL = $null
 	$newRes = $null
 	$newSize = $null
 	$page.Links | 
   		Where-Object { $_.href -like '*imgres*' } | 
   		ForEach-Object { ($_.href -split 'imgurl=')[-1].Split('&')[0]} |
   		ForEach-Object {
 			$u = $_	
 			$p = 0
 			while (($u.indexof("%") -ne -1) -and ($p -lt 10))
 			{
 				$p++
 				$u = [system.web.httputility]::UrlDecode($u)
 			}
     			try {
     				$req=[System.Net.HttpWebRequest]::Create($u)
     				$req.useragent=$browserAgent
 				$req.timeout = 30000
     				$res = $req.getresponse()
     				$res.close()
 			}
 			catch { $res = $null }
 			if (($res -ne $null) -and ($res.ContentLength -ne -1) -and ($res.ContentLength -gt $f.Length) -and ($res.statuscode -eq "OK"))
     			{
         			if (($newSize -eq $null) -or ($res.ContentLength -gt $newSize))
         			{
             				$newSize = $res.ContentLength
             				$newRes = $res
             				$newURL = $u
         			}
     			}
   		}
  
  		if ($newSize -eq $null) {"No bigger versions of $($f.basename)$($f.extension) found."; continue}
     		if ($newRes.responseuri.AbsoluteUri.lastindexof(".") -ne -1)
     		{
         		$ext = $newRes.responseuri.AbsoluteUri.substring($newRes.responseuri.AbsoluteUri.lastindexof("."))
         		if (($ext.length -gt 4) -and ($ext -ne ".jpeg"))
         		{
             			$ext=$ext.Substring(0,4)
         		}
     		}
     		else
     		{
         		$ext = ".jpg"
     		}
     		$f3 = $f.basename + $ext
     		$f3 = join-path $TargetFolder $f3    
     		del -ea 0 $f3
     		try 
     		{
        	 		Invoke-WebRequest -ea 0 -Uri $newURL -OutFile $f3 -UserAgent $browserAgent -TimeoutSec 30
     		}
     		catch { }
     		if (test-path -ea 0 $f3)
     		{
         		$f3 = get-item -ea 0 $f3
         		"Found $($f3.Length) byte file to replace $($f.Length) byte file: $($f.BaseName)$($f.extension)"
         		$foundfiles++
     		}
     		[gc]::collect()
 }
 
 "$($foundfiles) larger pictures found."
`

