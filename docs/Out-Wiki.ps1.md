---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 526
Published Date: 
Archived Date: 2010-01-25t15
---

# out-wiki - 

## Description

out-wiki – converts cmdlets help to media wiki (wikipedia) format

## Comments



## Usage



## TODO



## script

`out-wiki`

## Code

`#
 #
 ################################################################################
 ################################################################################
 ################################################################################
 
 function Out-Wiki {
    param($commands = $null, $outputDir = "./help")
 
    $commandsHelp = $commands | sort-object noun | get-help -full
 
    if ( -not (Test-Path $outputDir)) {
    		md $outputDir | Out-Null
    }
    
    $indexFileName = $outputDir + "/index.txt"
 
 
 @'
 The ActiveRoles Management Shell for Active Directory is an Active Directoryspecific
 automation and scripting shell that provides a command-line
 management interface for administering directory data either via Quest
 ActiveRoles Server or by directly accessing Active Directory domain
 controllers. The ActiveRoles Management Shell is built on Microsoft Windows
 PowerShell technology.
 
 The following cmdlets are currently in the package:
 
 '@  | out-file $indexFileName
 
    foreach ($c in $commandsHelp) {
       "* [[$($c.Name)]]"   | out-file $indexFileName -Append
    }
 
    '[[Category:QAD cmdlets reference]]'  | out-file $indexFileName -Append
    
    $outputText = $null
    foreach ($c in $commandsHelp) {
       $fileName = ( $outputDir + "/" + $c.Name + ".txt" )
 
 	  "$($c.synopsis)"  | out-file $fileName
       
       "== Syntax =="  | out-file $fileName -Append
       "<code>$(($c.syntax | out-string  -width 2000).Trim())</code>"  | out-file $fileName -Append
 
       "== Detailed Description =="  | out-file $fileName -Append
 	  "$($c.Description  | out-string  -width 2000)"  | out-file $fileName -Append
    
       "== Related Commands =="  | out-file $fileName -Append
       foreach ($relatedLink in $c.relatedLinks.navigationLink) {
          if($relatedLink.linkText -ne $null -and 
             $relatedLink.linkText.StartsWith("about") -eq $false){
             "* [[$($relatedLink.linkText)]]"  | out-file $fileName -Append         
          }
       }
    
       "== Parameters =="  | out-file $fileName -Append
 @'
 {| class="wikitable" border=1
 |-
 !Name
 !Description
 !Required?
 !Pipeline Input
 !Default Value
 '@   | out-file $fileName -Append
 
       $paramNum = 0
       foreach ($param in $c.parameters.parameter ) {
          "|-"  | out-file $fileName -Append
          "!$($param.Name)"  | out-file $fileName -Append
          "|$(($param.Description  | out-string  -width 2000).Trim())"  | out-file $fileName -Append
          "|$($param.Required)"  | out-file $fileName -Append
          "|$($param.PipelineInput)"  | out-file $fileName -Append
          "|$($param.DefaultValue)"  | out-file $fileName -Append
       }
       "|}"  | out-file $fileName -Append
    
 	  if (($c.inputTypes | Out-String ).Trim().Length -gt 0) {
 		"== Input Type =="  | out-file $fileName -Append
 		"$(($c.inputTypes  | out-string  -width 2000).Trim())"  | out-file $fileName -Append
 	  }
    
 	  if (($c.returnValues | Out-String ).Trim().Length -gt 0) {
 		"== Return Values =="  | out-file $fileName -Append
 		"$(($c.returnValues  | out-string  -width 2000).Trim())"  | out-file $fileName -Append
 	  }
 	  
 	 if (($c.alertSet | Out-String).Trim().Length -gt 0) {
       "== Notes =="  | out-file $fileName -Append
       "$(($c.alertSet  | out-string -Width 2000).Trim() )"  | out-file $fileName -Append
 	}
    
 	 if (($c.examples | Out-String).Trim().Length -gt 0) {
       "== Examples =="  | out-file $fileName -Append	  
 	  foreach ($example in $c.examples.example) {
 	      "==== $($example.title.Trim(('-',' ')))===="  | out-file $fileName -Append
 	      "<pre>$(($example.code | out-string ).Trim())</pre>"  | out-file $fileName -Append
 	      "$(($example.remarks | out-string -Width 2000).Trim())"  | out-file $fileName -Append
 	  	
 	  }
 	  }
 	  
       '[[Category:QAD cmdlets reference]]'   | out-file $fileName -Append
    }
 }
 
 Out-Wiki (Get-Command -PSSnapin Quest.ActiveRoles.ADManagement) "c:\Temp\QADHelp"
`

