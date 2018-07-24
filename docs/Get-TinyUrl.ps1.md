---
Author: alvaro torres
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6140
Published Date: 2016-12-18t00
Archived Date: 2016-09-09t00
---

# get-tinyurl - 

## Description

get a short url, using tiny-url api for shortening

## Comments



## Usage



## TODO



## function

`get-tinyurl`

## Code

`#
 #
 function Get-TinyUrl
 {
     [CmdletBinding()]
     [OutputType([string])]
     Param
     (
         [Parameter(Mandatory,ValueFromPipeline=$true)]
         [string]
         $Url,
 
         [ValidateSet('xml','json', 'text')]
         [string]
         $Format = 'text',
 
         [ValidateSet('0_mk', '0l_ro', '2u_lc', '3le_ru', '888_hn', '9mp_com', 'ad_vu', 'b54_in', 'bb-h_me', 'bim_im', 'bit_ly', 'chilp_it', 'clicky_me', 'cmprs_me', 'cort_as', 'crum_bs', 'curt_cc', 'cut_by', 'dfly_pk', 'dft_ba', 'di_gd', 'dlvr_it', 'dok_do', 'doo_ly', 'dssurl_com')]        
         [string]
         $Provider = '0_mk'
     )
 
     $SystemProxy = ([System.Net.WebProxy]::GetDefaultProxy().Address.AbsoluteUri)
     
     $TinyUrlApiKey = 'YOUR-APIKEY'
     $TinyUrlApiUrl = 'http://tiny-url.info/api/v1/create'
 
     $RequestUrl = 'http://tiny-url.info/api/v1/create?apikey={0}&provider={1}&format={2}&url={3}' -f $TinyUrlApiKey, $Provider, $Format, $Url
     Write-Debug $RequestUrl
    Invoke-RestMethod -Uri $RequestUrl -Method Get -Proxy $SystemProxy | Write-Output
 }
 
 #'https://msdn.microsoft.com/es-es/library/system.uri(v=vs.110).aspx' | Get-TinyUrl
`

