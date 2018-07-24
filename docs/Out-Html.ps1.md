---
Author: vegard hamar
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 587
Published Date: 2009-09-16t14
Archived Date: 2017-05-22t04
---

# out-html - 

## Description

out-html – converts cmdlets help to html format

## Comments



## Usage



## TODO



## script

`out-html`

## Code

`#
 #
 ################################################################################
 #
 ################################################################################
 ################################################################################
 
 param($outputDir = "./help")
 
 function FixString {
 	param($in = "")
 	if ($in -eq $null) {
 		$in = ""
 	}
 	return $in.Replace("&", "&amp;").Replace("<", "&lt;").Replace(">", "&gt;")
 }
 
 function Out-HTML {
 	param($commands = $null, $outputDir = "./help")
 
 	$commandsHelp = $commands | sort-object PSSnapIn, noun | get-help -full
 
 	if ( -not (Test-Path $outputDir)) {
 		md $outputDir | Out-Null
 	}
 
 	$indexFileName = $outputDir + "/index.htm"
 	
 @'
 <html>
 	<head>
 		<title>PowerShell Help</title>
 	</head>
 	<frameset cols="250,*">
 		<frame src="./index.htm" />
 		<frame src="" name="display"/>
 	</frameset>
 </html>
 '@ | Out-File "$outputDir/default.htm"
 
 @'
 <html>
 	<head>
 		<title>PowerShell Help</title>
 	</head>
 	<body>
 '@  | out-file $indexFileName
 
 	$SnapIn = ""
 	foreach ($c in $commandsHelp) {
 		if ($SnapIn -ne $c.PSSnapIn.Name) {
 			"<a href='#" + $c.PSSnapIn.name + "'>* " + $c.PSSnapIn.Name.Replace(".", " ") + "</a></br>"   | out-file $indexFileName -Append
 			$SnapIn = $c.PSSnapIn.Name
 		}
 	}
 
 	$SnapIn = ""
 	foreach ($c in $commandsHelp) {
 		if ($SnapIn -ne $c.PSSnapIn.Name) {
 			"<h3><a name='$($c.PSSnapIn.Name)'>" +$c.PSSnapIn.Name.Replace(".", " ") + "</a></h3>" | Out-File $indexFileName -Append
 			$SnapIn = $c.PSSnapIn.Name
 		}
 		"<a href='" + $c.name + ".htm' target='display'>* $($c.Name)</a></br>"   | out-file $indexFileName -Append
 	}
 
 	$outputText = $null
 	foreach ($c in $commandsHelp) {
 		$fileName = ( $outputDir + "/" + $c.Name + ".htm" )
 
 @"
 <html>
 	<head>
 		<title>$($c.Name)</title>
 	</head>
 	<body>
 		<h1>$($c.Name)</h1>
 		<div>$($c.synopsis)</div>
 
 		<h2> Syntax </h2>
 		<code>$(FixString($c.syntax | out-string  -width 2000).Trim())</code>  
 
 		<h2> Detailed Description </h2>
 		<div>$(FixString($c.Description  | out-string  -width 2000))</div>
 
 		<h2> Related Commands </h2>
 		<div>
 "@ | out-file $fileName 
 		foreach ($relatedLink in $c.relatedLinks.navigationLink) {
 			if($relatedLink.linkText -ne $null -and $relatedLink.linkText.StartsWith("about") -eq $false){
 				"			* <a href='$($relatedLink.linkText).htm'>$($relatedLink.linkText)</a><br/>" | out-file $fileName -Append         
 			}
 		}
 	  
 @"
 		</div>
 		<h2> Parameters </h2>
 		<table border='1'>
 			<tr>
 				<th>Name</th>
 				<th>Description</th>
 				<th>Required?</th>
 				<th>Pipeline Input</th>
 				<th>Default Value</th>
 			</tr>
 "@   | out-file $fileName -Append
 
 		$paramNum = 0
 		foreach ($param in $c.parameters.parameter ) {
 @"
 			<tr valign='top'>
 				<td>$($param.Name)&nbsp;</td>
 				<td>$(FixString(($param.Description  | out-string  -width 2000).Trim()))&nbsp;</td>
 				<td>$(FixString($param.Required))&nbsp;</td>
 				<td>$(FixString($param.PipelineInput))&nbsp;</td>
 				<td>$(FixString($param.DefaultValue))&nbsp;</td>
 			</tr>
 "@  | out-file $fileName -Append
 		}
 		"		</table>}"  | out-file $fileName -Append
    
 		if (($c.inputTypes | Out-String ).Trim().Length -gt 0) {
 @"
 		<h2> Input Type </h2>
 		<div>$(FixString($c.inputTypes  | out-string  -width 2000).Trim())</div>
 "@  | out-file $fileName -Append
 		}
    
 		if (($c.returnValues | Out-String ).Trim().Length -gt 0) {
 @"
 		<h2> Return Values </h2>
 		<div>$(FixString($c.returnValues  | out-string  -width 2000).Trim())</div>
 "@  | out-file $fileName -Append
 		}
           
 		if (($c.alertSet | Out-String).Trim().Length -gt 0) {
 @"
 		<h2> Notes </h2>
 			"<div>$(FixString($c.alertSet  | out-string -Width 2000).Trim())</div>
 "@  | out-file $fileName -Append
 		}
    
 		if (($c.examples | Out-String).Trim().Length -gt 0) {
 			"		<h2> Examples </h2>"  | out-file $fileName -Append      
 			foreach ($example in $c.examples.example) {
 @"
 		<h3> $(FixString($example.title.Trim(('-',' '))))</h3>
 				<pre>$(FixString($example.code | out-string ).Trim())</pre>
 				<div>$(FixString($example.remarks | out-string -Width 2000).Trim())</div>
 "@  | out-file $fileName -Append
 			}
 		}
 @"
 	</body>
 </html>
 "@ | out-file $fileName -Append
 	}
 @"
 	</body>
 </html>
 "@ | out-file $indexFileName -Append
 }
 
 Out-HTML (Get-Command ) $outputDir
`

