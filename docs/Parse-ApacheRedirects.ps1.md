---
Author: anonymous
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4076
Published Date: 2014-04-05t16
Archived Date: 2017-03-24t16
---

# parse-apacheredirects - 

## Description

parses an apache redirect file to identify broken entries.

## Comments



## Usage



## TODO



## script

`invoke-resourcerequest`

## Code

`#
 #
 <#
 .SYNOPSIS
     Parses an Apache redirect file to identify broken entries.
 .DESCRIPTION
     All redirects will be followed and the final response code will be processed as follows:
         200 - OK
         401 - OK
         403 - Broken
         404 - Broken
         *** - Optionally remove with -RemoveUnknownErrors switch
 
     Only Errors are written to the screen which include 403, 404, and unknown errors.
     If more verbosity is desired set the $VerbosePreference variable as follows:
     $VerbosePreference = 'Continue'
 
     Written for a Windows 8 system and hasn't been tested on anything else.
 .PARAMETER RedirectFile
     Location of the Apache redirect file.
 .PARAMETER WebsiteDomain
     Domain name of the website that uses the redirect file.
 .PARAMETER CreateNewFile
     Will create a new redirect file by appending ".Broken_Removed" to the file name.
 
     Unknown Errors will NOT be removed unless the -RemoveUnknownErrors switch is used.
 .PARAMETER RemoveUnknownErrors
     Can only be used if creating a new redirect file as it will 
     comment out Unknown Errors
 .EXAMPLE
     PS D:\> .\Parse-ApacheRedirects.ps1 -RedirectFile D:\redirects.conf -WebsiteDomain website.domain.com -CreateNewFile -RemoveUnknownErrors
     Line 2: ERROR Redirect 301 /somefile http://website.domain.com/newlocation.html
     Line 3: ERROR Redirect 301 /anotherfile https://website2.domain.com/
     Line 18: UNKNWN_ERROR: Redirect 301 /foo http://nothing.domain.com/
     Line 18: SVR_RSPND:System.Net.WebException: The remote name could not be resolved: 'nothing.domain.com'
     Line 18: Removing_From_New_Conf
 
     New file available here:  D:\redirects.conf.Broken_Removed
 
         Finished
 .NOTES
     As mentioned in the Description, setting $VerbosePreference='Continue' will enable
     all Write-Verbose commands in the script.
 
     **This script was written on a Windows 8 system and hasn't been tested on anything else.**
 #>
 
 Param(
     [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()][ValidateScript({Test-Path $_})][string]$RedirectFile,
     [Parameter(Mandatory=$true)][ValidateNotNullOrEmpty()][ValidateScript({Resolve-DnsName $_})][string]$WebsiteDomain,
     [switch]$CreateNewFile,
     [switch]$RemoveUnknownErrors
     )
 
 Function Invoke-ResourceRequest($Uri,[switch]$ReturnSC){
     $RequestedWebsite = "$WebsiteDomain$Uri"
     try{
         $Response = Invoke-WebRequest -Uri $RequestedWebsite
         if($Response.StatusCode -eq 200){ 
             if($ReturnSC){ Return 200 }
             else{ Return 0 }
         }
         else{ Return 1 }
     }
     Catch { 
         [string]$eResponse = $_.Exception.ToString().split("`n")[0]
         if($ReturnSC){ Return $eResponse }
         else{
             if($eResponse.Contains("(404) Not Found")){ Return 1 }
             elseif($eResponse.Contains("(403) Forbidden")){ Return 1 }
             elseif($eResponse.Contains("(401) Unauthorized")){ Return 0 }
             else{ Return 2 }
         }
     }
 
 Function Parse-Uri([string]$LineObject,[int]$LineNumber){
     if($LineObject.Contains("`"")){
         $quotes = $LineObject.split("`"").Count
         if($quotes -eq 3){
             if($LineObject.IndexOf("`"") -gt 13){ 
                 Write-Verbose "Line $LineNumber`: Destination_Url_Quoted "
                 Return $LineObject.Split(" ")[2]
             }
             else{ 
                 Write-Verbose "Line $LineNumber`: Uri_Quoted "
                 Return $LineObject.Split("`"")[1]
             }
         }
         elseif($quotes -eq 5){
             Write-Verbose "Line $LineNumber`: Uri_and_Destination_Url_Quoted "
             Return $LineObject.Split("`"")[1]
         }
         else{ 
             Write-Host "Line $LineNumber`: QUOTE_ERROR_REMOVE_RULE" -ForegroundColor Cyan
             Write-Verbose "Line $LineNumber`: Should_Only_Have_2_or_4_Quotes"
             Return ""
         }
     }
     else{
         Write-Verbose "Line $LineNumber`: Normal "
         Return $LineObject.Split(" ")[2]
     }
 
 $RedirectContent = Get-Content -Path $RedirectFile
 
 if($CreateNewFile){ 
     $NewFile = (Get-Item -Path $RedirectFile).FullName+".Broken_Removed" 
     if(Test-Path -Path $NewFile){
         if((Read-Host "New conf file exists. Overwrite? [y/n]").ToLower() -eq "y"){ Clear-Content -Path $NewFile }
         else { 
             Write-Host "`n`tRename $NewFile and launch script again or choose `"y`" to overwrite.`n" 
             Exit
         }
     }
 }
 
 if((!($CreateNewFile)) -and ($RemoveUnknownErrors)){
     Write-Host "`n`tMust include the `"CreateNewFile`" switch if removing unknown errors.`n`tExiting script.`n" -ForegroundColor Red
     Exit
 }
 
 $i=1
 foreach($line in $RedirectContent){
     [string]$RequestUri = ""
     [int]$Check = 0
     if($line.StartsWith("#")){ Write-Verbose "Line $i`: Comment" }
     else{
         if($line.Contains("Redirect")){
             if($line.Substring(9,1) -eq "3"){
                 $RequestUri = Parse-Uri -LineObject $line -LineNumber $i
             }
                 $RequestUri = Parse-Uri -LineObject $line.tolower().Replace("redirect ","redirect 000 ") -LineNumber $i
             }
             if($RequestUri -eq ""){ $Check = 1 }
         }
         elseif($line.Length -le 3){
             Write-Verbose "Line $i`: Empty"
         }
         else{
             Write-Verbose "Line $i`: NOT_REDIRECT_OR_EMPTY $line"
             Write-Verbose "Line $i`: Rule_Was_NOT_Removed "
         }
     }
 
     if($RequestUri){ 
         Write-Verbose "Line $i`: Uri_found_making_request"
         $Check = Invoke-ResourceRequest -Uri $RequestUri 
     }
 
     if($Check -eq 2){ 
         Write-Verbose "Line $i`: Destination_Returned_an_Error "
         Write-Verbose "Line $i`: Making_another_request_to_parse_response "
         $ResponseCode = Invoke-ResourceRequest -Uri $RequestUri -ReturnSC
         Write-Host "Line $i`: UNKNWN_ERROR`: $line" -ForegroundColor Magenta
         Write-Host "Line $i`: SVR_RSPND`:$ResponseCode" -ForegroundColor Magenta
         if($RemoveUnknownErrors -and $CreateNewFile){ 
             Write-Host "Line $i`: Removing_From_New_Conf " -ForegroundColor Magenta
             $Check = 2
         }
         else{ $Check = $0 }
     }
     if($Check -eq 1){ Write-Host "Line $i`: ERROR $line" -ForegroundColor Red }
 
     if($NewFile -ne $null){
         if($Check -eq 0){ Out-File -FilePath $NewFile -InputObject $line -Append -Encoding ascii }
     }
 
     $i++
 }
 if($NewFile){ Write-Host "`nNew file available here: "$NewFile -ForegroundColor Green }
 Write-Host "`nFinished`n" -ForegroundColor Green
 Exit
`

