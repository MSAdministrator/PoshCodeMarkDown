---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 4.3
Encoding: ascii
License: cc0
PoshCode ID: 5736
Published Date: 2015-02-14t23
Archived Date: 2015-03-23t13
---

# invoke-web - 

## Description

preview – (still) unfinished work ;-)

## Comments



## Usage



## TODO



## function

`convertto-dictionary`

## Code

`#
 #
 function ConvertTo-Dictionary {
   param(
     [Parameter(Mandatory=$true,ValueFromPipeline=$true,ParameterSetName="Hashtable")]
     [Hashtable[]]$Hashtable,
 
     [Parameter(Mandatory=$true,ValueFromPipelineByPropertyName=$true,ParameterSetName="WebHeaders")]
     [System.Collections.Specialized.NameObjectCollectionBase]$Headers,
 
     [Parameter(Mandatory=$true,ParameterSetName="Hashtable")]
     [Type]$TKey,
 
     [Parameter(Mandatory=$true,ParameterSetName="Hashtable")]
     [Type]$Tvalue
   )
   begin {
     switch($PSCmdlet.ParameterSetName) {
       "Hashtable" {
         $dictionary = New-Object "System.Collections.Generic.Dictionary[[$($TKey.FullName)],[$($TValue.FullName)]]" 
       }
       "WebHeaders" {
         $dictionary = New-Object "System.Collections.Generic.Dictionary[[String],[String]]" 
       }
     }
   }
   process { 
     switch($PSCmdlet.ParameterSetName) {
       "Hashtable" {
         foreach($ht in $Hashtable) { 
           foreach($key in $ht.Keys) { 
             $dictionary.Add( $key, $ht.$key ) 
           }
         }
       }
       "WebHeaders" {
         foreach($key in $Headers.AllKeys) {
           $dictionary.Add($key, $Headers[$key])
         }
       }
     }
   }
   end { return $dictionary }
 }
 function ConvertFrom-Dictionary {
   [CmdletBinding()]
   param($Dictionary, [Switch]$Encode)
   foreach($key in $Dictionary.Keys) {
     "{0} = {1}" -f $key, $( if($Encode) { [System.Net.WebUtility]::UrlEncode( $Dictionary.$key ) } else { $Dictionary.$key } )
   }
 }
 
 function Invoke-Web {
   <#
     .Synopsis
       Downloads a file or page from the web, or sends web API posts/requests
     .Description
       Creates an HttpWebRequest to download a web file or post data
     .Example
       Invoke-Web http://PoshCode.org/PoshCode.psm1
     
       Downloads the latest version of the PoshCode module to the current directory
     .Example
       Invoke-Web http://PoshCode.org/PoshCode.psm1 ~\Documents\WindowsPowerShell\Modules\PoshCode\
     
       Downloads the latest version of the PoshCode module to the default PoshCode module directory...
     .Example
       $RssItems = @(([xml](Invoke-Web http://poshcode.org/api/ -passthru)).rss.channel.GetElementsByTagName("item"))
     
       Returns the most recent items from the PoshCode.org RSS feed
     .Notes
       History:
       v4.3  - Fixed stupid missing (or extra) characters. I don't know how...      
       v4.2  - Fixed bugs when sending content in the body.
             - Added -ForceBasicAuth to allow client-side to specify Authentication: Basic header.
       v4.1  - Reworked most of it with PowerShell 3's Invoke-WebRequest as inspiration
             - Added a bunch of parameters, the ability to do PUTs etc., and session/cookie persistence
             - Did NOT parse the return code and get you the FORMs the way PowerShell 3 does -- upgrade! ;)
       v3.12 - Added full help
       v3.9 - Fixed and replaced the Set-DownloadFlag
       v3.7 - Removed the Set-DownloadFlag code because it was throwing on Windows 7:
              "Attempted to read or write protected memory."
       v3.6.6 Add UserAgent calculation and parameter
       v3.6.5 Add file-name guessing and cleanup
       v3.6 - Add -Passthru switch to output TEXT files 
       v3.5 - Add -Quiet switch to turn off the progress reports ...
       v3.4 - Add progress report for files which don't report size
       v3.3 - Add progress report for files which report their size
       v3.2 - Use the pure Stream object because StreamWriter is based on TextWriter:
              it was messing up binary files, and making mistakes with extended characters in text
       v3.1 - Unwrap the filename when it has quotes around it
       v3   - rewritten completely using HttpWebRequest + HttpWebResponse to figure out the file name, if possible
       v2   - adds a ton of parsing to make the output pretty
              added measuring the scripts involved in the command, (uses Tokenizer)
   #>
   [CmdletBinding(DefaultParameterSetName="NoSession")]
   param(
         [Parameter(Mandatory=$true,Position=0)]
         [System.Uri][Alias("Url")]$Uri,
 
         [Parameter(ValueFromPipeline=$true)]
         $Body,
 
         [String]$ContentType,
 
         [System.Security.Cryptography.X509Certificates.X509Certificate[]]
         $Certificate,
 
         [string]$OutFile,
 
         [Switch]$Unblocked,
         [switch]$Passthru,
         [switch]$Quiet,
         [Parameter(Mandatory=$true,ParameterSetName="CreateSession")]
         [String]$SessionVariable,
         [Parameter(Mandatory=$true,ParameterSetName="UseSession")]
         $WebSession,
         [switch]$UseDefaultCredentials,
         [System.Management.Automation.PSCredential]
         [System.Management.Automation.Credential()]
         [Alias("")]$Credential = [System.Management.Automation.PSCredential]::Empty,
 
         [ValidateScript({if(!($Credential -or $WebSession)){ throw "ForceBasicAuth requires the Credential parameter be set"} else { $true }})]
         $ForceBasicAuth,   
         $DisableKeepAlive,
         [System.Collections.IDictionary]$Headers,
         [int]$MaximumRedirection = 5,
         [ValidateSet("Default", "Delete", "Get", "Head", "Options", "Post", "Put", "Trace")]
         [String]$Method = "Get",
         [Uri]$Proxy,
         [switch]$ProxyUseDefaultCredentials,
         [System.Management.Automation.PSCredential]
         [System.Management.Automation.Credential()]
         $ProxyCredential= [System.Management.Automation.PSCredential]::Empty,
         [string]$UserAgent = "Mozilla/5.0 (Windows NT; Windows NT $([Environment]::OSVersion.Version.ToString(2)); $PSUICulture) WindowsPowerShell/$($PSVersionTable.PSVersion.ToString(2)); PoshCode/4.0; http://PoshCode.org"     
   )
 
   process {
     Write-Verbose "Downloading '$Uri'"
     $EAP,$ErrorActionPreference = $ErrorActionPreference, "Stop"
     $request = [System.Net.HttpWebRequest]::Create($Uri)
     if($DebugPreference -ne "SilentlyContinue") {
       Set-Variable WebRequest -Scope 2 -Value $request
     }
 
     $ErrorActionPreference = $EAP
     $request.Method = $Method.ToUpper()
 
     if($WebSession) {
       $request.CookieContainer = $WebSession.Cookies
       $request.Headers = $WebSession.Headers
       if($WebSession.UseDefaultCredentials) {
          $request.UseDefaultCredentials
       } elseif($WebSession.Credentials) {
          $request.Credentials = $WebSession.Credentials
       }
       $request.ClientCertificates = $WebSession.Certificates
       $request.UserAgent = $WebSession.UserAgent
       $request.Proxy = $WebSession.Proxy
       $request.MaximumAutomaticRedirections = $WebSession.MaximumRedirection
     } else {
       $request.CookieContainer = $Cookies = New-Object System.Net.CookieContainer
     }
    
     $request.UserAgent = $UserAgent
     $request.MaximumAutomaticRedirections = $MaximumRedirection
     $request.KeepAlive = !$DisableKeepAlive
 
 
     if($Certificate) {
       $request.ClientCertificates.AddRange($Certificate)
     }
     if($UseDefaultCredentials) {
       $request.UseDefaultCredentials = $true
     } elseif($Credential -ne [System.Management.Automation.PSCredential]::Empty) {
       $request.Credentials = $Credential.GetNetworkCredential()
     }
 
     if($Proxy) { $request.Proxy = New-Object System.Net.WebProxy $Proxy }
     if($request.Proxy -ne $null) {
       if($ProxyUseDefaultCredentials) {
         $request.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
       } elseif($ProxyCredentials -ne [System.Management.Automation.PSCredential]::Empty) {
         $request.Proxy.Credentials = $ProxyCredentials
       }
     }
 
     if($ForceBasicAuth) {
       if(!$request.Credentials) {
         throw "ForceBasicAuth requires Credentials!"
       }
       $request.Headers.Add('Authorization', 'Basic ' + [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes($request.Credentials.UserName+":"+$request.Credentials.Password )));
     }
 
     if($SessionVariable) {
       Set-Variable $SessionVariable -Scope 1 -Value $WebSession
     }
    
     if($Headers) {
       foreach($h in $Headers.Keys) {
         $request.Headers.Add($h, $Headers[$h])
       }
     }
 
     if($Body) {
       if($Body -is [System.Collections.IDictionary] -or $Body -is [System.Collections.Specialized.NameObjectCollectionBase]) {
         if(!$ContentType) {
           $ContentType = "application/x-www-form-urlencoded"
         }
         [String]$Body = ConvertFrom-Dictionary $Body -Encode:$($ContentType -eq "application/x-www-form-urlencoded")
       } else {
         $Body = $Body | Out-String
       }
 
       $encoding = New-Object System.Text.ASCIIEncoding
       $bytes = $encoding.GetBytes($Body);
       $request.ContentType = $ContentType
       $request.ContentLength = $bytes.Length
       $writer = $request.GetRequestStream();
       $writer.Write($bytes, 0, $bytes.Length)
       $writer.Close()
     }
 
     try {
       $response = $request.GetResponse();
       if($DebugPreference -ne "SilentlyContinue") {
         Set-Variable WebResponse -Scope 2 -Value $response
       }
     } catch [System.Net.WebException] { 
       Write-Error $_.Exception -Category ResourceUnavailable
       return
       Write-Error $_.Exception -Category NotImplemented
       return
     }
  
     Write-Verbose "Retrieved $($Response.ResponseUri)"
     if((Test-Path variable:response) -and $response.StatusCode -eq 200) {
       if($OutFile -and !(Split-Path $OutFile)) {
         $OutFile = Join-Path (Convert-Path (Get-Location -PSProvider "FileSystem")) $OutFile
       }
       elseif((!$Passthru -and !$OutFile) -or ($OutFile -and (Test-Path -PathType "Container" $OutFile)))
       {
         [string]$OutFile = ([regex]'(?i)filename=(.*)$').Match( $response.Headers["Content-Disposition"] ).Groups[1].Value
         $OutFile = $OutFile.trim("\/""'")
         
         $ofs = ""
         $OutFile = [Regex]::Replace($OutFile, "[$([Regex]::Escape(""$([System.IO.Path]::GetInvalidPathChars())$([IO.Path]::AltDirectorySeparatorChar)$([IO.Path]::DirectorySeparatorChar)""))]", "_")
         $ofs = " "
         
         if(!$OutFile) {
           $OutFile = $response.ResponseUri.Segments[-1]
           $OutFile = $OutFile.trim("\/")
           if(!$OutFile) { 
             $OutFile = Read-Host "Please provide a file name"
           }
           $OutFile = $OutFile.trim("\/")
           if(!([IO.FileInfo]$OutFile).Extension) {
             $OutFile = $OutFile + "." + $response.ContentType.Split(";")[0].Split("/")[1]
           }
         }
         $OutFile = Join-Path (Convert-Path (Get-Location -PSProvider "FileSystem")) $OutFile
       }
 
       if($Passthru) {
         $encoding = [System.Text.Encoding]::GetEncoding( $response.CharacterSet )
         [string]$output = ""
       }
  
       [int]$goal = $response.ContentLength
       $reader = $response.GetResponseStream()
       if($OutFile) {
         try {
           $writer = new-object System.IO.FileStream $OutFile, "Create"
           Write-Error $_.Exception -Category WriteError
           return
         }
       }
 
       [byte[]]$buffer = new-object byte[] 4096
       [int]$total = [int]$count = 0
       do {
         $count = $reader.Read($buffer, 0, $buffer.Length);
         Write-Verbose "Read $count"
         if($OutFile) {
           $writer.Write($buffer, 0, $count);
         } 
         if($Passthru){
           $output += $encoding.GetString($buffer,0,$count)
         } elseif(!$quiet) {
           $total += $count
           if($goal -gt 0) {
             Write-Progress "Downloading $Uri" "Saving $total of $goal" -id 0 -percentComplete (($total/$goal)*100)
           } else {
             Write-Progress "Downloading $Uri" "Saving $total bytes..." -id 0
           }
         }
       } while ($count -gt 0)
       
       $reader.Close()
       if($OutFile) {
         $writer.Flush()
         $writer.Close()
       }
       if($Passthru){
         $output
       }
     }
     if(Test-Path variable:response) { $response.Close(); }
     if($SessionVariable) {
       Set-Variable $SessionVariable -Scope 1 -Value ([PSCustomObject]@{
         Headers               = ConvertTo-Dictionary -Headers $request.Headers
         Cookies               = $response.Cookies
         UseDefaultCredentials = $request.UseDefaultCredentials
         Credentials           = $request.Credentials
         Certificates          = $request.ClientCertificates
         UserAgent             = $request.UserAgent
         Proxy                 = $request.Proxy
         MaximumRedirection    = $request.MaximumAutomaticRedirections
       })
     }
     if($WebSession) {
       $WebSession.Cookies      = $response.Cookies
     }
   }
 }
 
`

