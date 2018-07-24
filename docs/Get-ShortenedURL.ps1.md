---
Author: chris campbell
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3865
Published Date: 2013-01-07t20
Archived Date: 2013-01-10t19
---

# get-shortenedurl - 

## Description

a function that returns the actual url from a http redirect. accepts pipeline input but does nothing useful with errors.

## Comments



## Usage



## TODO



## function

`get-shortenedurl`

## Code

`#
 #
 Function Get-ShortenedURL {
  <#
 .SYNOPSIS
  
     Get-ShortenedURL
     
     Author: Chris Campbell (@obscuresec)
     License: BSD 3-Clause
     
 .DESCRIPTION
 
     A function that returns the actual URL from a http redirect.
 
 .PARAMETER $ShortenedURL
 
     Specifies the shortened URL.
 
 .EXAMPLE
 
     PS C:\> Get-ShortenedURL -ShortenedURL http://goo.gl/V4PKq
     PS C:\> Get-ShortenedURL -ShortenedURL http://goo.gl/V4PKq,http://bit.ly/IeWSIZ
     PS C:\> Get-Content C:\urls.txt | Get-ShortenedURL 
     PS C:\> Get-ShortenedURL -ShortenedURL (Get-Content C:\urls.txt)
 
 .LINK
 
     http://obscuresecurity.blogspot.com/2013/01/Get-ShortenedURL.html
     https://github.com/obscuresec/random/blob/master/Get-ShortenedURL
 
 #>
 
     [CmdletBinding()] Param(
             [Parameter(Mandatory=$True,ValueFromPipeline=$True)]             
             [string[]] $ShortenedURL 
             )
 
     BEGIN {}
         
     PROCESS {
 
        Try {
             Foreach ($URL in $ShortenedURL) {
                 
                 $WebClientObject = New-Object System.Net.WebClient
                 $WebRequest = [System.Net.WebRequest]::create($URL)
                 $WebResponse = $WebRequest.GetResponse()
                 
                 $ActualDownloadURL = $WebResponse.ResponseUri.AbsoluteUri
                 
                 $ObjectProperties = @{ 'Shortened URL' = $URL;
                                        'Actual URL' = $ActualDownloadURL}
                 $ResultsObject = New-Object -TypeName PSObject -Property $ObjectProperties
                 
                 Write-Output $ResultsObject
                 
                 $WebResponse.Close()
             }       
        }
 
         Catch {}
     }
 
     END {}
 }
`

