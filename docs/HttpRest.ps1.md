---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0.1
Encoding: ascii
License: cc0
PoshCode ID: 787
Published Date: 
Archived Date: 2010-05-08t05
---

# httprest - 

## Description

added get-webpagecontent to the initial implementation of the http rest script functions (usable as a v1 script or as a module).

## Comments



## Usage



## TODO



## module

`get-google`

## Code

`#
 #
 ####################################################################################################
 ##
 ##
 ####################################################################################################
 ####################################################################################################
 ##
 ####################################################################################################
 if(!$PSScriptRoot){ 
    Write-Debug $($MyInvocation.MyCommand | out-string)
    $PSScriptRoot=(Split-Path $MyInvocation.MyCommand.Path -Parent) 
 }
 
 Write-Debug "PSScriptRoot: '$PSScriptRoot'"
 
 
 $null = [Reflection.Assembly]::LoadFrom( "$PSScriptRoot\mindtouch.dream.dll" )
 $null = [Reflection.Assembly]::LoadWithPartialName("System.Web")
 
 [uri]$global:url = ""
 [System.Management.Automation.PSCredential]$global:HttpRestCredential = $null
 
 function Get-DreamMessage($Content,$Type) {
    if(!$Content) { 
       return [MindTouch.Dream.DreamMessage]::Ok()
    }
    if( $Content -is [System.Xml.XmlDocument]) {
       return [MindTouch.Dream.DreamMessage]::Ok( $Content )
    }
    
    if(Test-Path $Content -EA "SilentlyContinue") {
       return [MindTouch.Dream.DreamMessage]::FromFile((Convert-Path (Resolve-Path $Content))); 
    }
    if($Type -is [String]) {
       $Type = [MindTouch.Dream.MimeType]::$Type
    }
    if($Type -is [MindTouch.Dream.DreamMessage]) {
       return [MindTouch.Dream.DreamMessage]::Ok( $Type, $Content )
    } else {  
       return [MindTouch.Dream.DreamMessage]::Ok( $([MindTouch.Dream.MimeType]::TEXT), $Content )
    }
 }
 
 function Get-DreamPlug {
    PARAM ( $Url, [hashtable]$With )
    if($Url -is [array]) {
       if($Url[0] -is [hashtable]) {
          $plug = [MindTouch.Dream.Plug]::New($global:url)
          foreach($param in $url.GetEnumerator()) {
             if($param.Value) {
                $plug = $plug.At($param.Key,"=$(Encode-Twice $param.Value)")
             } else {
                $plug = $plug.At($param.Key)
             }
          }
       } 
       else 
       {
          [URI]$uri = Join-Url $global:url $url 
          $plug = [MindTouch.Dream.Plug]::New($uri)
       }
    }
    elseif($url -is [string]) 
    {
       [URI]$uri = $url
       if(!$uri.IsAbsoluteUri) {
          $uri = Join-Url $global:url $url
       }
       $plug = [MindTouch.Dream.Plug]::New($uri)
    } 
    else {
       $plug = [MindTouch.Dream.Plug]::New($global:url)
    }
    if($with) { 
       foreach($param in $with.GetEnumerator()) {
          if($param.Value) {
             $plug = $plug.With($param.Key,$param.Value)
          }
       } 
    }
    return $plug
 }
 
 Function Receive-Http {
 PARAM(
    $Output = "Xml" 
 , 
    [string]$Path
 ,
    $InputObject
 #,
 ) 
 BEGIN {
    if($InputObject) {
       Write-Output $inputObject | Receive-Http $Output $Path 
 }
 PROCESS {
    $response = $null
    if($_ -is [MindTouch.Dream.Result``1[[MindTouch.Dream.DreamMessage]]]) {
       $response = $_.Wait()
    } elseif($_ -is [MindTouch.Dream.DreamMessage]) {
       $response = $_
    } elseif($_) {
       throw "We can only pipeline [MindTouch.Dream.DreamMessage] objects, or [MindTouch.Dream.Result`1[DreamMessage]] objects"
    }
    
    if($response) {
       Write-Debug $($response | Out-String)
       if(!$response.IsSuccessful) {
          Write-Error $($response | Out-String)
          Write-Verbose $response.AsText()
          throw "ERROR: '$($response.Status)' Response Status."
       } else {   
          switch($Output) {
             "File" {
                if(!$Path) { 
                   [string]$fileName = ([regex]'(?i)filename=(.*)$').Match( $response.Headers["Content-Disposition"] ).Groups[1].Value
                   $Path = $fileName.trim("\/""'")
                   if(!$Path) {
                      $fileName = $response.ResponseUri.Segments[-1]
                      $Path = $fileName.trim("\/")
                      if(!([IO.FileInfo]$Path).Extension) {
                         $Path = $Path + "." + $response.ContentType.Split(";")[0].Split("/")[1]
                      }
                   }
                }
                
                $File = Get-FileName $Path
                [StreamUtil]::CopyToFile( $response.AsStream(), $response.ContentLength, $File )
                Get-ChildItem $File
             }
             "XDoc" {
                if($Path) { 
                   $response.AsDocument()[$Path]
                } else {
                   $response.AsDocument()#.ToXml()
                }
             }
             "Xml" {
                if($Path) { 
                   $response.AsDocument().ToXml().SelectNodes($Path)
                } else {
                   $response.AsDocument().ToXml()
                }
             }
             "Text" {
                if($Path) { 
                   $response.AsDocument()[$Path] | % { $_.AsText }
                } else {
                   $response.AsText()
                }
             }
             "Bytes" {
                $response.AsBytes()
             }
          }
       }
    }
 }
 }
 
 Function Invoke-Http {
 PARAM( 
    [string]$Verb = "Get"
 ,
    $Path
 , 
    [hashtable]$with
 , 
    $Content
 ,
 ,
    $authenticate
 , 
    [switch]$waitForResponse
 )
 BEGIN {
    $Verbs = "Post", "Get", "Put", "Delete", "Head", "Options"
    if($Verbs -notcontains $Verb) {
       Write-Warning "The specified verb '$Verb' is NOT one of the common verbs: $Verbs"
    }
 
    if($Path) {
       if($Content) {
          Write-Output ($Path | Invoke-Http $Verb -With $With -Content $Content -Type $Type -Authenticate $authenticate -waitForResponse:$WaitForResponse)
       } else {
          Write-Output ($Path | Invoke-Http $Verb -With $With -Type $Type-Authenticate $authenticate -waitForResponse:$WaitForResponse)
       }
 }
 PROCESS {
    if($_) {
       $Path = $_
       
       $plug = Get-DreamPlug $Path $With
       Write-Verbose "Content Type: $Type"
       Write-Verbose "Content: $Content"
       if($Type -like "Form*" -and !$Content) {
          $dream = [MindTouch.Dream.DreamMessage]::Ok( $Plug.Uri )
          $Plug = [MindTouch.Dream.Plug]::New( $Plug.Uri.SchemeHostPortPath )
          Write-Verbose "RECREATED Plug: $($Plug.Uri.SchemeHostPortPath)"
       } else {
          $dream = Get-DreamMessage $Content $Type
       }
       
       if(!$plug -or !$dream) {
          throw "Can't come up with a request!"
       }
       
       Write-Verbose $( $dream | Out-String )
       
       if($authenticate){ 
             Write-Verbose "AUTHENTICATE AS $($authenticate.UserName)"
          if($authenticate -is [System.Management.Automation.PSCredential]) {
             Write-Verbose "AUTHENTICATING AS $($authenticate.UserName)"
             $plug = $plug.WithCredentials($authenticate.UserName, ($authenticate.GetNetworkCredential().Password))
          } elseif($authenticate -is [System.Net.NetworkCredential]) {
             Write-Verbose "AUTHENTICATING AS $($authenticate.UserName)"
             $plug = $plug.WithCredentials($authenticate.UserName, $authenticate.Password)
          } else {
             Get-HttpCredential
             Write-Verbose "AUTHENTICATING AS $($authenticate.UserName)"
             $plug = $plug.WithCredentials($global:HttpRestCredential.UserName, $global:HttpRestCredential.Password)
          }
       }
       
       Write-Verbose $plug.Uri
       
       Write-Debug "URI: $($Plug.Uri)"
       Write-Debug "Verb: $($Verb.ToUpper())"
       Write-Debug $($dream | Out-String)
       
       $result = $plug.InvokeAsync( $Verb.ToUpper(),  $dream )
       
       Write-Debug $($result | Out-String)
       
       if($waitForResponse) { 
          $result = $result.Wait() 
       }
       
       
       write-output $result
    
       trap [MindTouch.Dream.DreamResponseException] {
          Write-Error @"
 TRAPPED DreamResponseException
       
 $($_.Exception.Response | Out-String)
 
 $($_.Exception.Response.Headers | Out-String)
 "@
          break;
       }
    }
 }
 }
 
 
 function Get-WebPageContent {
 #[CmdletBinding()]
 param( 
    [string]$url
 ,
    [string]$xpath=""
 ,
    [hashtable]$with=@{}
 ,
    [switch]$AsXml 
 )
 BEGIN { $out = "Text"; if($AsXml) { $out="Xml" } }
 PROCESS {
    invoke-http get $url $with | receive-http $out $xpath
 }
 }
 
 new-alias gwpc Get-WebPageContent -EA "SilentlyContinue"
 new-alias http Invoke-Http        -EA "SilentlyContinue"
 new-alias rcv  Receive-Http       -EA "SilentlyContinue"
 
 
 
 function Set-HttpDefaultUrl {
 PARAM ([uri]$baseUri=$(Read-Host "Please enter the base Uri for your RESTful web-service"))
    $global:url = $baseUri 
 }
 
 function Set-HttpCredential {
    param($Credential=$(Get-CredentialBetter -Title   "Http Authentication Request - $($global:url.Host)" `
                                       -Message "Your login for $($global:url.Host)" `
                                       -Domain  $($global:url.Host)) )
    if($Credential -is [System.Management.Automation.PSCredential]) {
       $global:HttpRestCredential = $Credential
    } elseif($Credential -is [System.Net.NetworkCredential]) {
       $global:HttpRestCredential = new-object System.Management.Automation.PSCredential $Credential.UserName, $(ConvertTo-SecureString $credential.Password)
    }
 }
 
 function Get-HttpCredential([switch]$Secure) {
    if(!$global:url) { Set-HttpDefaultUrl }
    if(!$global:HttpRestCredential) { Set-HttpCredential }
    if(!$Secure) {
       return $global:HttpRestCredential.GetNetworkCredential();
    } else {
       return $global:HttpRestCredential
    }
 }
 
 
 
 function Encode-Twice {
    param([string]$text)
    return [System.Web.HttpUtility]::UrlEncode( [System.Web.HttpUtility]::UrlEncode( $text ) )
 }
 
 function Join-Url ( [uri]$baseUri=$global:url ) {
    $ofs="/";$BaseUrl = ""
    if($BaseUri -and $baseUri.AbsoluteUri) {
       $BaseUrl = "$($baseUri.AbsoluteUri.Trim('/'))/"
    }
    return [URI]"$BaseUrl$([string]::join("/",@($args)).TrimStart('/'))"
 }
 
 function ConvertTo-SecureString {
 Param([string]$input)
    $result = new-object System.Security.SecureString
 
    foreach($c in $input.ToCharArray()) {
       $result.AppendChar($c)
    }
    $result.MakeReadOnly()
    return $result
 }
 
 function Get-FileName {
    param($fileName=$([IO.Path]::GetRandomFileName()), $path)
    $fileName = $fileName.trim("\/""'")
    if($Path -and !(Test-Path $Path -Type Container) -and (Test-Path (Split-Path $path) -Type Container)) {
       $path
    } elseif($Path -and (Test-Path $path -Type Container)) {
       $fileName = Split-Path $fileName -leaf
       Join-Path $path $fileName
    } elseif((Split-Path $fileName) -and (Test-Path (Split-Path $fileName))) {
       $fileName
    } else {
       Join-Path (Get-Location -PSProvider "FileSystem") (Split-Path $fileName -Leaf)
    }
 }
 
 function Get-UtcTime {
    Param($Format="yyyyMMddhhmmss")
    [DateTime]::Now.ToUniversalTime().ToString($Format)
 }
 
 ###################################################################################################
 ###################################################################################################
 function Get-CredentialBetter{ 
 PARAM([string]$UserName, [string]$Title, [string]$Message, [string]$Domain, [switch]$Console)
    $ShellIdKey = "HKLM:\SOFTWARE\Microsoft\PowerShell\1\ShellIds"
    $cp = (Get-ItemProperty $ShellIdKey ConsolePrompting -ea "SilentlyContinue")
    $cp = $cp.ConsolePrompting -eq $True
 
    if($Console -and !$cp) {
       Set-ItemProperty $ShellIdKey ConsolePrompting $True
    } elseif(!$Console -and $Console.IsPresent -and $cp) {
       Set-ItemProperty $ShellIdKey ConsolePrompting $False
    }
 
    $Host.UI.PromptForCredential($Title,$Message,$UserName,$Domain,"Generic","Default")
 
    Set-ItemProperty $ShellIdKey ConsolePrompting $cp
 }
`

