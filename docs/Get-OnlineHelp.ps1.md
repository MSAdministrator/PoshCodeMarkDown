---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.3
Encoding: ascii
License: cc0
PoshCode ID: 1724
Published Date: 2012-03-24t21
Archived Date: 2012-07-26t00
---

# get-onlinehelp - 

## Description

retrieve online cmdlet help — fix a type check

## Comments



## Usage



## TODO



## function

`select-expand`

## Code

`#
 #
 
 
 
 param(
    [Parameter(Mandatory=$true,Position=0)]
    [String]$Name
 ,
    [Parameter(Mandatory=$false,Position=1)]
    [String[]]$Sections= @("Name", "Synopsis", "Syntax", "Description")
 )
 
 
 function Select-Expand {
 .Synopsis
    Like Select-Object -Expand, but with recursive iteration of a select chain
 .Description
    Takes a dot-separated series of properties to expand, and recursively iterates the output of each property ...
 .Parameter Property
    A collection of property names to expand.
    
    Each property can be a dot-separated series of properties, and each one is expanded, iterated, and then evaluated against the next
 .Parameter InputObject
    The input to be selected from
 .Parameter Unique
    If set, this becomes a pipeline hold-up, and the total output is checked to ensure uniqueness
 .Parameter EmptyToo
    If set, Select-Expand will include empty/null values in it's output
 .Example
    Get-Help Get-Command | Select-Expand relatedLinks.navigationLink.uri -Unique
 
    This will return the online-help link for Get-Command.  It's the equivalent of running the following command:
 
    C:\PS> Get-Help Get-Command | Select-Object -Expand relatedLinks | Select-Object -Expand navigationLink | Select-Object -Expand uri | Where-Object {$_} | Select-Object -Unique
 #>
 param(
    [Parameter(ValueFromPipeline=$false,Position=0)]
    [string[]]$Property
 ,
    [Parameter(ValueFromPipeline=$true)]
    [Alias("IO")]
    [PSObject[]]$InputObject
 ,
    [Switch]$Unique
 ,
    [Switch]$EmptyToo
 )
 begin { 
    if($unique) {
       $output = @()
    }
 }
 process {
    foreach($io in $InputObject) {
       foreach($prop in $Property -split "\.") {
          if($io -ne $null) {
             $io = $io | Select-Object -Expand $prop
             Write-Verbose $($io | out-string)
          }
       }
       if(!$EmptyToo -and ($io -ne $null)) {
          $io = $io | Where-Object {$_}
       }
       if($unique) {
          $output += @($io)
       } 
       else {
          Write-Output $io
       }
    }
 }
 end {
    if($unique) {
       Write-Output $output | Select-Object -Unique
    }
 }
 }
 
 function Get-HttpResponseUri {
 #.Synopsis
 #.Description
 #.Parameter ShortUrl
    PARAM(
       [Parameter(Position=0,Mandatory=$true,ValueFromPipeline=$true)]
       [Alias("Uri","Url")]
       [string]$ShortUrl
    )
    $req = [System.Net.HttpWebRequest]::Create($ShortUrl)
    $req.Method = "HEAD"
    $response = $req.GetResponse()
    Write-Output $response.ResponseUri
 }
 
 if( -not("Mtps.ContentService" -as [type] -and $global:MtpsWebServiceProxy -as "Mtps.ContentService")) {
 $global:MtpsWebServiceProxy = New-WebServiceProxy "http://services.msdn.microsoft.com/ContentServices/ContentService.asmx?wsdl" -Namespace Mtps
 }
 
 function Get-OnlineHelpContent {
 param(
    [Parameter(Mandatory=$true,Position=0)]
    [String]$Name
 ,
    [Parameter(Mandatory=$false,Position=1)]
    [String[]]$Sections= @("Name", "Synopsis", "Syntax", "Description")
 )
 process { 
    $uri = Get-Help $Name | Select-Expand relatedLinks.navigationLink.uri -Unique | Get-HttpResponseUri
    
    if(!$uri) { throw "Couldn't find online help URL for $Name" }
    
    $id = [IO.Path]::GetFileNameWithoutExtension( $uri.segments[-1] )
    write-verbose "Content Id: $id"
 
    $content = $MtpsWebServiceProxy.GetContent( (New-Object 'Mtps.getContentRequest' -Property @{locale = $PSUICulture; contentIdentifier = $id; requestedDocuments = (New-Object Mtps.requestedDocument -Property @{Selector="Mtps.Failsafe"}) }) )
    $global:OnlineHelpContent = $content.primaryDocuments |?{$_.primaryFormat -eq "Mtps.Failsafe"} | Select -Expand Any
    
    $NameNode = $global:OnlineHelpContent.SelectSingleNode("//*[local-name()='div' and @class='topic']/*[local-name()='div' and @class='title']")
    $NameNode.SetAttribute("header","NAME")
    
    $global:OnlineHelpContent = $global:OnlineHelpContent.SelectSingleNode("//*[local-name()='div' and @id='mainBody']")
    
    $Synopsis = $global:OnlineHelpContent.SelectSingleNode("*[local-name()='p']")
    $Synopsis.SetAttribute("header","SYNOPSIS")
    
    $headers = $OnlineHelpContent.h2  | ForEach-Object { $_.get_InnerText().Trim() }
    $content = $OnlineHelpContent.div | ForEach-Object { $_ }
 
    $global:help = @{Name=$NameNode; Synopsis=$Synopsis}
    if($headers.Count -ne $content.Count) { 
       Write-Warning "Unmatched content"
       foreach($header in $headers) {
         $help.$header = $OnlineHelpContent.SelectNodes( "//*[preceding-sibling::*[1][local-name()='h2' and normalize-space()='$header']]" )
         $help.$header.SetAttribute("header",$header.ToUpper())
       }
    }
    else {
       for($h=0;$h -lt $headers.Count; $h++){
          $help.($headers[$h]) = $content[$h]
          $help.($headers[$h]).SetAttribute("header",$headers[$h].ToUpper())
       }
    }
    $help
    
    $content[$Sections] | ForEach-Object { 
       Write-Output
       Write-Output $_.Header
       Write-Output
       Write-Output ($_.get_InnerText() -replace '^[\n\s]*\n|\n\s+$')
    }
 }
 }
 
 Get-OnlineHelpContent $Name | out-null
 
 
 $help[$Sections] | ForEach-Object { 
    Write-Host
    Write-Host $_.Header -Fore Cyan
    Write-Host
    Write-Host ($_.get_InnerText() -replace '^[\n\s]*\n|\n\s+$')
 }
`

