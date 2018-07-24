---
Author: sean kearney
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5704
Published Date: 2017-01-22t20
Archived Date: 2017-03-25t01
---

# create rtf - 

## Description

functions to create an rtf file only using windows powershell

## Comments



## Usage



## TODO



## script

`write-rtfdoc`

## Code

`#
 #
 $Filename='.\test.rtf' 
  
 Function Write-RTFDoc 
 { 
 Param($content)  
  
 $OutputFile=$Script:Filename 
  
 Add-Content -Value $content -Path $Outputfile -Force 
 } 
  
 Function Write-RTFLine 
 { 
 param ($Style,$Content) 
  
     switch ($Style) 
     { 
         '0' { $output="{\rtf1\ansi\ansicpg1252\deff0\nouicompat\deflang1033{\fonttbl{\f0\fnil\fcharset0 Calibri;}{\f1\fnil\fcharset2 Symbol;}}`r`n{\*\generator Riched20 6.3.9600}\viewkind4\uc1 "} 
         '1' { $output=$Content+"\par"} 
         '2' { $output="\pard{\pntext\f1\'B7\tab}{\*\pn\pnlvlblt\pnf1\pnindent0{\pntxtb\'B7}}\fi-360\li720\sl276\slmult1\f0\fs22\lang9 "+$Content+"\par" } 
         '3' { $output="{\pntext\f1\'B7\tab}"+$content+"\par`r`n\pard\sl276\slmult1\par"} 
         '4' { $output = "\par" } 
         '99' { $output="}" }         
         '999' { $output="\pard\sl276\slmult1\f0\fs22\lang9 "} 
     '9999' { $output="\pard\sl276\slmult1\qc\ul\b\f0\fs40 "+$Content+"\par`r`n\pard\sl276\slmult1\fs22\ulnone\b0\par" } 
      }                     
     Write-RTFDoc $output 
  
 } 
  
 Function Write-RTFColumn 
 { 
 param($Style,$value1,$value2,$value3,$value4) 
          
      switch($Style) 
     {    
         '0' { $output="\trowd\trgaph108\trleft5\trbrdrl\brdrs\brdrw10 \trbrdrt\brdrs\brdrw10 \trbrdrr\brdrs\brdrw10 \trbrdrb\brdrs\brdrw10 \trpaddl108\trpaddr108\trpaddfl3\trpaddfr3 
 \clbrdrl\brdrw10\brdrs\clbrdrt\brdrw10\brdrs\clbrdrr\brdrw10\brdrs\clbrdrb\brdrw10\brdrs \cellx3121\clbrdrl\brdrw10\brdrs\clbrdrt\brdrw10\brdrs\clbrdrr\brdrw10\brdrs\clbrdrb\brdrw10\brdrs \cellx6238\clbrdrl\brdrw10\brdrs\clbrdrt\brdrw10\brdrs\clbrdrr\brdrw10\brdrs\clbrdrb\brdrw10\brdrs \cellx9355  
 \pard\intbl\widctlpar\f1\fs22 "+$value1+"\cell "+$value2+"\cell "+$value3+"\cell\row  
 \pard\sa200\sl276\slmult1\f2\lang9" } 
         '1' { $output="\trowd\trgaph108\trleft5\trbrdrl\brdrs\brdrw10 \trbrdrt\brdrs\brdrw10 \trbrdrr\brdrs\brdrw10 \trbrdrb\brdrs\brdrw10 \trpaddl108\trpaddr108\trpaddfl3\trpaddfr3 
 \clbrdrl\brdrw10\brdrs\clbrdrt\brdrw10\brdrs\clbrdrr\brdrw10\brdrs\clbrdrb\brdrw10\brdrs \cellx2342\clbrdrl\brdrw10\brdrs\clbrdrt\brdrw10\brdrs\clbrdrr\brdrw10\brdrs\clbrdrb\brdrw10\brdrs \cellx4679\clbrdrl\brdrw10\brdrs\clbrdrt\brdrw10\brdrs\clbrdrr\brdrw10\brdrs\clbrdrb\brdrw10\brdrs \cellx7017\clbrdrl\brdrw10\brdrs\clbrdrt\brdrw10\brdrs\clbrdrr\brdrw10\brdrs\clbrdrb\brdrw10\brdrs \cellx9355  
 \pard\intbl\widctlpar\f1\fs22 "+$value1+"\cell "+$value2+"\cell "+$value3+"\cell "+$value4+"\cell\row  
 \pard\sa200\sl276\slmult1\f2\lang9" } 
     } 
     Write-RTFDoc $output 
  
 } 
  
`

