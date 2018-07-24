---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.1
Encoding: ascii
License: cc0
PoshCode ID: 2376
Published Date: 2012-11-19t22
Archived Date: 2012-11-30t03
---

# poweroauth - 

## Description

this is the second release, but still very raw. supports oauth via installed application authentication (a modified form of oauth where the consumer fetches access tokens using a username and password instead of a request token) ... now includes a sample app for fetching stuff off yammer.

## Comments



## Usage



## TODO



## module

`get-oauthbase`

## Code

`#
 #
 Set-StrictMode -Version 2.0
 $null = [Reflection.Assembly]::LoadWithPartialName('System.Web')
 $safeChars = [char[]]'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-_.~'
 
 function Get-OAuthBase {
 #.Synopsis
 #.Parameter ConsumerKey
 #.Parameter AccessToken
 #.Parameter AccessVerifier
 #.ReturnValue
 PARAM( 
    [Parameter(Mandatory=$true)]$ConsumerKey
 ,  [Parameter(Mandatory=$false)]$AccessToken 
 ,  [Parameter(Mandatory=$false)]$AccessVerifier
 ,  [Parameter(Mandatory=$false)]
    [ValidateSet("HMAC-SHA1", "PLAINTEXT")]
    [String]
    $SignatureMethod = "HMAC-SHA1"
 )
 END {
    @{ 
       oauth_consumer_key      = Format-OAuth $ConsumerKey
       oauth_timestamp         = [int]((Get-Date).ToUniversalTime() - (Get-Date 1/1/1970)).TotalSeconds
       oauth_nonce             = [guid]::NewGuid().GUID -replace '-'
    } + $(if($AccessToken){ @{oauth_token = $AccessToken } } else { @{} }
 }}
 
 function Format-OAuth {
 #.Synopsis 
 #.Parameter value
 PARAM([Parameter(ValueFromPipeline=$true)][string]$value)
 PROCESS {
    $result = New-Object System.Text.StringBuilder
    foreach($c in $value.ToCharArray()){
       if($safeChars -notcontains $c) {
          $null = $result.Append( ('%{0:X2}' -f [int]$c) )
       } else {
          $null = $result.Append($c)
       }
    }
    write-output $result.ToString()
 }}
 
 function ConvertTo-Hashtable {
 #.Synopsis
 #.Parameter string
 #.Parameter PairSeparator
 #.Parameter KeyValueSeparator
 #.ReturnValue
 PARAM(
    [Parameter(ValueFromPipeline=$true, Position=0, Mandatory=$true)]
    [String]
    $string
 ,
    [Parameter(Position=1)]
    [String]
    $PairSeparator = '&'
 ,  
    [Parameter(Position=2)]
    [String]
    $KeyValueSeparator = '='
 )
 PROCESS {
    $result = @{}
    foreach($w in $string.split($pairSeparator) ) {
       $k,$v = $w.split($KeyValueSeparator,2)
       $result.Add($k,$v)
    }
    write-output $result
 }
 }
 
 function ConvertFrom-Hashtable {
 #.Synopsis
 #.Description
 #.Parameter values
 #.Parameter PairSeparator
 #.Parameter KeyValueSeparator
 #.ReturnValue
 PARAM(
    [Parameter(ValueFromPipeline=$true, Position=0, Mandatory=$true)]
    [Hashtable]
    $Values
 ,
    [Parameter(Position=1)]
    [String]
    $pairSeparator = '&'
 ,  
    [Parameter(Position=2)]
    [String]
    $KeyValueSeparator = '='
 ,
    [string[]]$Sort
 )
 PROCESS {
    [string]::join($pairSeparator, @(
       if($Sort) {
          foreach( $kv in $Values.GetEnumerator() | Sort $Sort) {
             if($kv.Name) {
                '{0}{1}{2}' -f $kv.Name, $KeyValueSeparator, $kv.Value
             }
          }
       } else {
          foreach( $kv in $Values.GetEnumerator()) {
             if($kv.Name) {
                '{0}{1}{2}' -f $kv.Name, $KeyValueSeparator, $kv.Value
             }
          }
       }
    ))
 }}
 
 function ConvertTo-OAuthSignatureBase {
 #.Synopsis
 PARAM(
    [Parameter(Mandatory=$true)]
    [Uri]
    $Uri
 ,
    [Parameter(Mandatory=$false)]
    [hashtable]
    $Parameters =@{}
 ,
    [Parameter(Mandatory=$false)]
    [ValidateSet("POST", "GET", "PUT", "DELETE", "HEAD", "OPTIONS")]
    [string]
    $Verb = "GET"
 )
 BEGIN {
    trap { continue }
    if(!$Uri.IsAbsoluteUri) {
       $Uri= Join-Url $global:url $Uri
       Write-Verbose "Relative URL, appending to $($global:url) to get: $Uri"
    }
 }
 END {
       $normalizedUrl = ("{0}://{1}" -f $Uri.Scheme, $Uri.Host).ToLower()
       if (!(($Uri.Scheme -eq "http" -and $Uri.Port -eq 80) -or ($Uri.Scheme -eq "https" -and $Uri.Port -eq 443)))
       {
           $normalizedUrl += ":" + $Uri.Port
       }
       
       $normalizedUrl += $Uri.AbsolutePath
       write-output $normalizedUrl
 
       if($Uri.Query) { 
          $Parameters += $(ConvertTo-Hashtable $Uri.Query.trim("?"))
       }
       $normalizedRequestParameters = Format-OAuth (ConvertFrom-Hashtable $Parameters -Sort "Name","Value")
       
       write-output $normalizedRequestParameters
       
       $result = New-Object System.Text.StringBuilder
       $null = $result.AppendFormat("{0}&", $verb.ToUpper() )
       $null = $result.AppendFormat("{0}&", $(Format-OAuth $normalizedUrl) )
       $null = $result.AppendFormat("{0}", $normalizedRequestParameters)
 
       write-output $result.ToString();
    }
 }
 
 function Get-OAuthSignature {
 #.Synopsis
 #.Parameter Verb
 #.Parameter Uri
 #.Parameter Parameters
 #.Parameter ConsumerKey
 #.Parameter ConsumerSecret
 #.Parameter AccessToken
 #.Parameter ConsumerSecret
 PARAM( 
    [Parameter(Mandatory=$true)]
    [Uri]
    $Uri
 ,
    [Parameter(Mandatory=$false)]
    [hashtable]
    $Parameters =@{}
 ,
    [Parameter(Mandatory=$true)]
    [string]
    $ConsumerKey = "key"
 , 
    [Parameter(Mandatory=$true)]
    [string]
    $ConsumerSecret = "secret"
 , 
    [Parameter(Mandatory=$false)]
    [string]
    $AccessToken = ""
 , 
    [Parameter(Mandatory=$false)]
    [string]
    $AccessSecret = ""
 , 
    [Parameter(Mandatory=$false)]
    [string]
    $AccessVerifier = ""
 , 
    [Parameter(Mandatory=$false)]
    [ValidateSet("POST", "GET", "PUT", "DELETE", "HEAD", "OPTIONS")]
    [string]
    $Verb = "GET"
 ,
    [ValidateSet("HMAC-SHA1", "PLAINTEXT")]
    [String]
    $SignatureMethod = "HMAC-SHA1"
 )
 END {
    @($Parameters.Keys) | % {
       $Parameters.$_ = @($Parameters.$_) | %{ Format-OAuth $_ }
    }
 
    $Parameters += Get-OAuthBase $ConsumerKey $AccessToken $AccessVerifier -SignatureMethod $SignatureMethod
 
    $url, $query, $sigbase = ConvertTo-OAuthSignatureBase -Uri $Uri -Parameters $Parameters -Verb $Verb 
    Write-Verbose ([System.Web.HttpUtility]::URLDecode( $sigbase ) -split "&" -join "`n")
 
    Write-Output $url, $Parameters
    if($SignatureMethod -eq "HMAC-SHA1") {
       $sha = new-object System.Security.Cryptography.HMACSHA1
       $sha.Key =  [System.Text.Encoding]::Ascii.GetBytes( ('{0}&{1}' -f $(Format-OAuth $ConsumerSecret),$(Format-OAuth $AccessSecret)) )
       Write-Output $([Convert]::ToBase64String( $sha.ComputeHash( [System.Text.Encoding]::Ascii.GetBytes( $sigbase ) ) ))
    } else {
       Write-Output ('{0}&{1}' -f $(Format-OAuth $ConsumerSecret),$(Format-OAuth $AccessSecret)) 
    }
 }}
 
 function Invoke-OAuthHttp {
 #.Synopsis
 #.Parameter Uri
 #.Parameter Parameters
 #.Parameter ConsumerKey
 #.Parameter ConsumerSecret
 #.Parameter AccessToken
 #.Parameter ConsumerSecret
 #.Parameter Verb
 [CmdletBinding()]
 PARAM(
    [Parameter(Mandatory=$true)]
    [Uri]
    $Uri
 ,
    [Parameter(Mandatory=$false)]
    [HashTable]
    $Parameters =@{}
 ,
    [Parameter(Mandatory=$true)]
    [string]
    $ConsumerKey = "key"
 , 
    [Parameter(Mandatory=$true)]
    [string]
    $ConsumerSecret = "secret"
 , 
    [Parameter(Mandatory=$false)]
    [string]
    $AccessToken = ""
 , 
    [Parameter(Mandatory=$false)]
    [string]
    $AccessSecret = ""
 ,
    [Parameter(Mandatory=$false)]
    [string]
    $AccessVerifier = ""
 , 
    [Parameter(Mandatory=$false)]
    [ValidateSet("POST", "GET", "PUT", "DELETE", "HEAD", "OPTIONS")]
    [string]
    $Verb = "GET"
 ,
    [ValidateSet("HMAC-SHA1", "PLAINTEXT")]
    [String]
    $SignatureMethod = "HMAC-SHA1"
 , 
    [ValidateSet("Xml", "File", "Text","Bytes")]
    [Alias("As")]
    $Output = "Xml"
 ,
    [AllowEmptyString()]
    [string]$Path
 )
 END {
    $parameters.format = "xml"
    
 
    if($AccessToken -and $AccessSecret) {
       $script:url, $script:Headers, $script:sig = Get-OAuthSignature -Uri $Uri -Parameters $Parameters `
                                                 -ConsumerKey    $ConsumerKey    `
                                                 -ConsumerSecret $ConsumerSecret `
                                                 -AccessToken    $AccessToken    `
                                                 -AccessSecret   $AccessSecret   `
                                                 -AccessVerifier $AccessVerifier `
                                                 -Verb $Verb.ToUpper()           `
                                                 -SignatureMethod $SignatureMethod
    } else {
       $script:url, $script:Headers, $script:sig = Get-OAuthSignature -Uri $Uri -Parameters $Parameters `
                                                 -ConsumerKey    $ConsumerKey    `
                                                 -ConsumerSecret $ConsumerSecret `
                                                 -Verb $Verb.ToUpper()           `
                                                 -SignatureMethod $SignatureMethod
    }
    
    $Headers += @{ oauth_signature = Format-OAuth $sig }
    $Parameters.Keys | %{ $Headers.Remove($_) }
    $WithHeader = @{ Authorization="OAuth {0}`"" -f $(ConvertFrom-Hashtable $Headers "`", " "=`"") }
 
    Write-Verbose $( $WithHeader | fl | Out-String )
    
    switch($Verb) {
       "POST" {
          $plug = Get-DreamPlug $Uri.AbsoluteUri -Headers $WithHeader
          $global:debugplug = $plug
          $script:result = Invoke-Http "POST" -Plug $plug -Content ([HashTable]$Parameters) | Receive-Http $Output -Path $Path
       }
       "GET" {
          $plug = Get-DreamPlug $Uri.AbsoluteUri -Headers $WithHeader -With $Parameters
          $global:debugplug = $plug
          $script:result = Invoke-Http "GET" -Plug $plug | Receive-Http $Output -Path $Path
       }
    }
       
    if(!$AccessToken -and $result -is [System.Xml.XmlDocument] -and $result.SelectNodes("html") -and $result.SelectNodes("html") -match "oauth_token_secret") {
       $result.html | ConvertTo-Hashtable
    } elseif($result -is [System.Xml.XmlDocument]){
       $result.SelectSingleNode("//*")
    } else {
       $result
    }
 }
 }
 
 #
 #
 #
 #
 
 
 
 
 
 
 
 ####################################################################################################
 $global:PoshOAuthToken = @{ 
 }
 $global:OAuthUris = @{
    AccessToken  = "https://www.yammer.com/oauth/access_token"
    RequestToken = "https://www.yammer.com/oauth/request_token"
    Authorize    = "https://www.yammer.com/oauth/access_token"
 }
 
 function Get-OAuthAccessToken {
 [CmdletBinding()]
 param()
 #.Synopsis
    if(!(Test-Path Variable:Global:OAuthAccessToken) -or !$global:OAuthAccessToken) {
    
       $RequestToken = Invoke-OAuthHttp                            `
         -Uri             $OAuthUris.RequestToken                  `
         -ConsumerKey     $PoshOAuthToken.oauth_consumer_key       `
         -ConsumerSecret  $PoshOAuthToken.oauth_consumer_secret    `
         -Verb "POST" -SignatureMethod 'HMAC-SHA1'
    
       if(!$RequestToken) { return }
       
       $Global:DebugToken = $RequestToken
       Write-Warning "Starting browser for interactive authorization"
       $AuthURL = ("https://www.yammer.com/oauth/authorize?oauth_token={0}" -f $RequestToken.oauth_token)
       Start $AuthURL
       
       $AccessVerifier = Read-Host "Please enter the token from the web site: $AuthURL"
 
       $global:OAuthAccessToken = Invoke-OAuthHttp                           `
         -Uri             $OAuthUris.AccessToken                   `
         -ConsumerKey     $PoshOAuthToken.oauth_consumer_key       `
         -ConsumerSecret  $PoshOAuthToken.oauth_consumer_secret    `
         -AccessToken     $RequestToken.oauth_token            `
         -AccessSecret    $RequestToken.oauth_token_secret     `
         -AccessVerifier  $AccessVerifier                          `
         -Verb "POST"
    }
    return $global:OAuthAccessToken
 }
 
 
 function Invoke-OAuthorized {
 [CmdletBinding()]
 PARAM(
    [Parameter(Mandatory=$true)]
    [Uri]
    $Uri
 ,
    [Parameter(Mandatory=$false,ValueFromPipeline=$true)]
    [hashtable]
    $Parameters =@{}
 ,
    [Parameter(Mandatory=$false)]
    [IO.FileInfo[]]
    $File
 ,
    [Parameter(Mandatory=$false)]
    [ValidateSet("POST", "GET", "PUT", "DELETE", "HEAD", "OPTIONS")]
    [string]
    $Verb = "GET"
 , 
    [ValidateSet("Xml", "File", "Text","Bytes")]
    [Alias("As")]
    $Output = "Xml"
 ,
    [AllowEmptyString()]
    [string]$Path
 )
 BEGIN {
    if(!(Test-Path Variable:Global:OAuthAccessToken) -or !$global:OAuthAccessToken) {
       Write-Warning "Must fetch new Access Token"
       $Global:OAuthAccessToken = Get-OAuthAccessToken
    }
 }
 PROCESS {
    Write-Verbose "Fetching URI: $Uri"
    Invoke-OAuthHttp -Parameters $Parameters -Uri $Uri                            `
                     -ConsumerKey    $Global:PoshOAuthToken.oauth_consumer_key    `
                     -ConsumerSecret $Global:PoshOAuthToken.oauth_consumer_secret `
                     -AccessToken    $Global:OAuthAccessToken.oauth_token         `
                     -AccessSecret   $Global:OAuthAccessToken.oauth_token_secret  `
                     -Verb $Verb -Output $Output -Path $Path
 }}
 
 
 function Get-AccessTokenForIAA {
 #.Synopsis
 param($Credential)
    if(!(Test-Path Variable:Global:IAAccessToken)) {
       if(!$Credential) { $Credential = Get-Credential }
       
       $global:IAAccessToken = Invoke-OAuthHttp -Parameters @{
       } -Uri             $FriendFeedUris.AccessToken `
         -ConsumerKey     $global:PoshFFToken.oauth_consumer_key `
         -ConsumerSecret  $global:PoshFFToken.oauth_consumer_secret -Verb "POST"
    }   
    return $global:IAAccessToken
 }
 
 
 
 
`

