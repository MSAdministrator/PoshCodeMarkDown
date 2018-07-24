---
Author: dirk bremen
Publisher: 
Copyright: 
Email: 
Version: 1.4.2
Encoding: ascii
License: cc0
PoshCode ID: 6499
Published Date: 2017-09-01t11
Archived Date: 2017-04-19t06
---

# invoke-jquery - 

## Description

invokes jquery (or plain javascript) commands via internetexplorer.application com object,after initial injection of jquery reference in header section.

## Comments



## Usage



## TODO



## function

`invoke-jquery`

## Code

`#
 #
 function Invoke-JQuery
 {
 <#
     .SYNOPSIS
         	Function to Invoke JQuery commands via IE COM
         
     .DESCRIPTION
 	Invokes JQuery (or plain Javascript) commands via InternetExplorer.Application COM object,
 	after initial injection of JQuery reference in header section. 
 	Useful to utilize JQuery selectors and functions for IE automation.
     
     .PARAMETER IE
         	Initialized InternetExplorer.Application COM object
     
     .PARAMETER Command
         	JQuery/Javascript to Invoke
 		
     .PARAMETER Function
         	Function to add in header section
         
     .PARAMETER Initialize
         	Switch to initially inject JQuery in header section.
         
     .EXAMPLE  
 	$ie = new-object -com internetexplorer.application
 	$ie.visible = $true
 	$ie.navigate2("http://www.example.com")
 	while($ie.busy) {start-sleep 1}
 	$div="<div id='myResult'>"
 	$ie.Document.body.innerHTML += $div
 	Invoke-JQuery $ie '$("a").hide();' -Initialize
 	$result = $ie.document.getElementById("myResult").innerHtml
 	$jFunc=@"
 	function SelectText(element) { 
 		var text = document.getElementById(element); 
 		var range = document.body.createTextRange(); 
 		range.moveToElementText(text); 
 		range.select(); 
 	}
 	"@
 	Invoke-JQuery $ie -Function $jFunc
 	Invoke-JQuery $ie 'SelectText("myResult");'  
 #>
     [cmdletbinding()]
     param(
         [parameter(position=0, mandatory=$true)]
         $IE,
 		
         [parameter(position=1,mandatory=$false)]
         $Command,
 		
         [parameter()]
         $Function,
         
         [parameter()]
         [switch]$Initialize
     )
 	if ($Initialize -or $Function){
 		$url='http://code.jquery.com/jquery-1.4.2.min.js'
 		$document = $IE.document 
 		$head = @($document.getElementsByTagName("head"))[0] 
 		$script = $document.createElement('script') 
 		$script.type = 'text/javascript'
 	}
     
 	if ($Initialize){
 		$script.src = $url 
 		$head.appendChild($script) | Out-Null
 	}
 
 	if ($Command){$IE.document.parentWindow.execScript("$Command","javascript")}
 
 	if ($Function){
 		$script.text = $Function
 		$head.appendChild($script) | Out-Null
 	}
 }
`

