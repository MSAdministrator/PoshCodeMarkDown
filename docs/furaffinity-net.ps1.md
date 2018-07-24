---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6282
Published Date: 
Archived Date: 2016-11-18t00
---

#  - 

## Description

furaffinity.net

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 workflow Send-Tweet {
     param (
     [Parameter(Mandatory=$true)][string]$Message
     )
 
     InlineScript {      
         [Reflection.Assembly]::LoadWithPartialName("System.Security")  
         [Reflection.Assembly]::LoadWithPartialName("System.Net")  
         
         $status = [System.Uri]::EscapeDataString($Using:Message);  
         $oauth_consumer_key = "AeotYQENRTSZ3lh4GOIOJ7tvT";  
         $oauth_consumer_secret = "<Redacted>";
         $oauth_token = "<Redacted>";
         $oauth_token_secret = "<Redacted>";
         $oauth_nonce = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes([System.DateTime]::Now.Ticks.ToString()));  
         $ts = [System.DateTime]::UtcNow - [System.DateTime]::ParseExact("01/01/1970", "dd/MM/yyyy", $null).ToUniversalTime();  
         $oauth_timestamp = [System.Convert]::ToInt64($ts.TotalSeconds).ToString();  
   
         $signature = "POST&";  
         $signature += [System.Uri]::EscapeDataString("https://api.twitter.com/1.1/statuses/update.json") + "&";  
         $signature += [System.Uri]::EscapeDataString("oauth_consumer_key=" + $oauth_consumer_key + "&");  
         $signature += [System.Uri]::EscapeDataString("oauth_nonce=" + $oauth_nonce + "&");   
         $signature += [System.Uri]::EscapeDataString("oauth_signature_method=HMAC-SHA1&");  
         $signature += [System.Uri]::EscapeDataString("oauth_timestamp=" + $oauth_timestamp + "&");  
         $signature += [System.Uri]::EscapeDataString("oauth_token=" + $oauth_token + "&");  
         $signature += [System.Uri]::EscapeDataString("oauth_version=1.0&");  
         $signature += [System.Uri]::EscapeDataString("status=" + $status);  
   
         $signature_key = [System.Uri]::EscapeDataString($oauth_consumer_secret) + "&" + [System.Uri]::EscapeDataString($oauth_token_secret);  
   
         $hmacsha1 = new-object System.Security.Cryptography.HMACSHA1;  
         $hmacsha1.Key = [System.Text.Encoding]::ASCII.GetBytes($signature_key);  
         $oauth_signature = [System.Convert]::ToBase64String($hmacsha1.ComputeHash([System.Text.Encoding]::ASCII.GetBytes($signature)));  
   
         $oauth_authorization = 'OAuth ';  
         $oauth_authorization += 'oauth_consumer_key="' + [System.Uri]::EscapeDataString($oauth_consumer_key) + '",';  
         $oauth_authorization += 'oauth_nonce="' + [System.Uri]::EscapeDataString($oauth_nonce) + '",';  
         $oauth_authorization += 'oauth_signature="' + [System.Uri]::EscapeDataString($oauth_signature) + '",';  
         $oauth_authorization += 'oauth_signature_method="HMAC-SHA1",'  
         $oauth_authorization += 'oauth_timestamp="' + [System.Uri]::EscapeDataString($oauth_timestamp) + '",'  
         $oauth_authorization += 'oauth_token="' + [System.Uri]::EscapeDataString($oauth_token) + '",';  
         $oauth_authorization += 'oauth_version="1.0"';  
     
         $post_body = [System.Text.Encoding]::ASCII.GetBytes("status=" + $status);   
         [System.Net.HttpWebRequest] $request = [System.Net.WebRequest]::Create("https://api.twitter.com/1.1/statuses/update.json");  
         $request.Method = "POST";  
         $request.Headers.Add("Authorization", $oauth_authorization);  
         $request.ContentType = "application/x-www-form-urlencoded";  
         $body = $request.GetRequestStream();  
         $body.write($post_body, 0, $post_body.length);  
         $body.flush();  
         $body.close();  
         $response = $request.GetResponse();
     }
  }
 do {
 	do {
 		$no = Get-Random -minimum 1 -maximum 3200050
 		$fauri = "http://www.furaffinity.net/view/" + $no
 		$site = Invoke-WebRequest -UseBasicParsing -Uri $fauri
 		$site.content | Out-File "%systemroot%\<Redacted>"
 		$checkcontent = Get-Content "%systemroot%\<Redacted>"
 		$checkcomment = $checkcontent | Select-String 'System Error' -Quiet
 		$stringfind = Select-String -pattern "replyto-message" -path "%systemroot%\<Redacted>" -context 0,1
 		$comment = $stringfind.context | select -Expand PostContext
 		$sexycomment = $comment[0] -split "<br/><br/>"
 		$sexycomment = $sexycomment -replace "!",""
 		$sexycomment = $sexycomment -replace "<br/>",""
 		$sexycomment = $sexycomment -replace "`'",""
 		$sexycomment | Out-File "%systemroot%\<Redacted>" -Append
 		Remove-Item "%systemroot%\<Redacted>"
 		sleep 5
 	}
 	until ($checkcomment -ne $true -and (($sexycomment | out-string).length -gt 3))
 
 	Send-Tweet ($sexycomment | Out-String)
 	$sexycomment = ""
 	sleep (Get-Random -minimum 180 -maximum 600)
 }
 until ($null)
 {}
`

