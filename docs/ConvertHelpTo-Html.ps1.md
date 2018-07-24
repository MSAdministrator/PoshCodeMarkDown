---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.0
Encoding: ascii
License: cc0
PoshCode ID: 670
Published Date: 2009-11-14t00
Archived Date: 2015-05-09t23
---

# converthelpto-html - 

## Description

convertto-dekicontent is an improvement on new-htmlhelp, with a specific focus on output suitable for dekiwiki.  i commented out the html and body tags, etc. because the html markup is destined for the dekiwiki.  it’s an improvement over the original because we cleaned up the parameter and example code, and broke apart the syntax, so it’s easier to read.

## Comments



## Usage



## TODO



## function

`get-htmlhelp`

## Code

`#
 #
 ####################################################################################################
 ####################################################################################################
 ##
 ##
 ##
 ####################################################################################################
 ####################################################################################################
 
 [System.Reflection.Assembly]::LoadWithPartialName("System.Web") | out-null
 
 function Get-HtmlHelp {
    param([string[]]$commands, [string]$baseUrl)
    $commands | Get-Command -type Cmdlet -EA "SilentlyContinue" | get-help -Full | ConvertTo-DekiContent $baseUrl
 }
 
 function ConvertTo-DekiContent {
 param($baseUrl)
 PROCESS {
       $help = $_
       
    
       $data += "<h2>Synopsis</h2>$($help.Synopsis | Out-HtmlPara)"
       
       $data += "<h2>Syntax</h2>$($help.Syntax | Out-HtmlPara)"
    
       $data += "<h2>Related Commands</h2>"
       foreach ($relatedLink in $help.relatedLinks.navigationLink) {
          if($relatedLink.linkText -ne $null -and $relatedLink.linkText.StartsWith("about") -eq $false) {
             $uri = ""
             if( $relatedLink.uri -ne "" ) {
                $uri = $relatedLink.uri
             } else{
                $uri = "$baseUrl/$((get-command $relatedLink.linkText -EA "SilentlyContinue").PSSnapin.Name)/$($relatedLink.linkText)"
             }
             $data += "<a href='$(encode($uri)).html'>$(encode($relatedLink.linkText))</a><br>"
          }
       }
    
       $data += "<h2>Detailed Description</h2>$(encode(&{$help.Description | out-string -width 200000}))"
    
       $data += "<h2>Parameters</h2>"
       $help.parameters.parameter | %{
          $param = $_
          $data += "<h4>-$(encode($param.Name)) [&lt;$(encode($param.type.name))&gt;]</h4>"
          $data += $param.Description | Out-HtmlPara
          $data += "<table>"
          $data += "<tr><th>Required? &nbsp;</th><td> $(encode($param.Required))</td></tr>"
          $data += "<tr><th>Position? &nbsp;</th><td> $(encode($param.Position))</td></tr>"
          $data += "<tr><th>Default value? &nbsp;</th><td> $(encode($param.defaultValue))</td></tr>"
          $data += "<tr><th>Accept pipeline input? &nbsp;</th><td> $(encode($param.pipelineInput))</td></tr>"
          $data += "<tr><th>Accept wildcard characters? &nbsp;</th><td> $(encode($param.globbing))</td></tr></table>"
       }
    
       if($help.inputTypes) {
          $data += "<h3>Input Type</h3>$($help.inputTypes | Out-HtmlPara)"
       }
       if($help.returnValues) {
          $data += "<h3>Return Type</h3>$($help.returnValues | Out-HtmlPara)"
       }
       $data += "<h2>Notes</h2>$($help.alertSet | Out-HtmlPara)"
    
       $data += "<h2>Examples</h2>"
       
       $help.Examples.example | %{
          $example = $_
          $data += "<h4>$(encode($example.title.trim(' -')))</h4>"
          $data += "<code><strong>PS&gt;</strong>&nbsp;$(encode($example.code))</code>"
          $data += "<p>$($example.remarks | out-string -width ([int]::MaxValue) | Out-HtmlPara)</p>"
 
       }
 
       write-output $data
    } else { 
       Write-Error "Can only process -Full view help output"
    }
 }}
 
 
 
 function encode($str) {
    begin{ if($str){ $str.split("`n") | encode  } }
    process{ if($_){ [System.Web.HttpUtility]::HtmlEncode($_).Trim() } }
 }
 
 function trim($str) {
    begin{ if($str){ $str.Trim() } }
    process{ if($_){ $_.Trim() } }
 }
 
 function split($Separator="`n",$inputObject) {
    begin{ if($inputObject){ $inputObject | split $Separator } }
    process{ if($_){ [regex]::Split($_,$Separator) | ? {$_.Length} } }
 }
 
 function join($Separator=$ofs,$inputObject) {
    begin{ if($inputObject){ [string]::Join($Separator,$inputObject) } else { $array =@() }}
    process{ if($_){ $array += $_ } }
    end{ if($array.Length) { [string]::Join($Separator,$array) } }
 }
 
 function Out-HtmlPara {
    process{if($_){"<p>$($_ | out-string -width ([int]::MaxValue) | split "\s*`n" | encode | trim | join "</p>`n<p>")</p>"}}
 }
`

