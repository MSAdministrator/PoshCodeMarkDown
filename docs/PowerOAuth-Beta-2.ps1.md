---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2377
Published Date: 2012-11-19t22
Archived Date: 2012-12-29t04
---

# poweroauth beta 2 - 

## Description

the beginnings of a complete rewrite … i’m actually going to recreate the functionality (but not the api) of my httprest module based on hammock … this is just the first function.

## Comments



## Usage



## TODO



## function

`get-oauthcredentials`

## Code

`#
 #
 if(@(Import-ConstructorFunctions -Path "$PSScriptRoot\Types_Generated").Count -lt 3) {
 Add-ConstructorFunction  -T Hammock.Authentication.OAuth.OAuthCredentials, Hammock.RestClient, Hammock.RestRequest -Path "$PSScriptRoot\Types_Generated"
 }
 
 $RequestToken = "request_token"
 $AccessToken   = "access_token"
 $Authorize     = "authorize"
 
 
 
 
 
 function Get-OAuthCredentials {
 #.Synopsis
 #.Description
 [CmdletBinding()]
 param(
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [string]$Authority                              
 ,                                          
    [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias('Key')]                          
    [string]$ConsumerKey                            
 ,                                          
    [Parameter(Position=2, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias('Secret')]
    [string]$ConsumerSecret
 )
 process {
    $CachePath = Join-Path $PSScriptRoot "$(($Authority -as [Uri]).Authority).clixml"
    if(Test-Path $CachePath) {
       return Import-CliXml $CachePath
    }
 
    ##############################################
    $cred1   = New-OAuthCredentials -Type RequestToken -SignatureMethod HmacSha1 -ConsumerKey $ConsumerKey -ConsumerSecret $ConsumerSecret
    $client  = New-RestClient -Cred $cred1 -Authority $Authority
    $request = New-RestRequest -Path $RequestToken
 
    $RequestResponse = $client.Request( $request )
 
    ##############################################
    $RequestAuth = [System.Web.HttpUtility]::ParseQueryString($RequestResponse.Content)
    $AuthURL =  "{0}/{1}?oauth_token={2}" -f $Authority.TrimEnd('/'), $Authorize, $RequestAuth['oauth_token']
    Start $AuthURL
    $AccessVerifier = Read-Host "Please enter the token from the web site: $AuthURL"
 
    ##############################################
    $cred2   = New-OAuthCredentials -Type AccessToken -SignatureMethod HmacSha1 -Verifier $AccessVerifier `
                  -ConsumerKey $ConsumerKey -ConsumerSecret $ConsumerSecret `
                  -Token  $RequestAuth['oauth_token'] -TokenSecret $RequestAuth['oauth_token_secret'] 
 
    $client  = New-RestClient -Cred $cred2 -Authority $Authority
    $request = New-RestRequest -Path $AccessToken
 
    $AccessResponse = $client.Request( $request )
    $AccessAuth     = [System.Web.HttpUtility]::ParseQueryString($AccessResponse.Content)
 
    ##############################################
    $credentials    = New-OAuthCredentials -Type AccessToken -SignatureMethod HmacSha1 `
                      -ConsumerKey $ConsumerKey -ConsumerSecret $ConsumerSecret `
                      -Token  $RequestAuth['oauth_token'] -TokenSecret $RequestAuth['oauth_token_secret'] 
 
    Export-CliXml $CachePath -Input $Credentials
    return $credentials
 }}
`

