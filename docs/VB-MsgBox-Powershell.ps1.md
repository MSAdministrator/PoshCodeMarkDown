---
Author: dyspareunia
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3419
Published Date: 2014-05-20t16
Archived Date: 2014-03-19t08
---

# vb msgbox powershell - 

## Description

this is an advanced function i found that i’ve been using as a show-msgbox function. enjoy

## Comments



## Usage



## TODO



## script

`show-msgbox`

## Code

`#
 #
             .SYNOPSIS  
             Shows a graphical message box, with various prompt types available. 
  
             .DESCRIPTION 
             Emulates the Visual Basic MsgBox function.  It takes four parameters, of which only the prompt is mandatory 
  
             .INPUTS 
             The parameters are:- 
              
             Prompt (mandatory):  
                 Text string that you wish to display 
                  
             Title (optional): 
                 The title that appears on the message box 
                  
             Icon (optional).  Available options are: 
                 Information, Question, Critical, Exclamation (not case sensitive) 
                 
             BoxType (optional). Available options are: 
                 OKOnly, OkCancel, AbortRetryIgnore, YesNoCancel, YesNo, RetryCancel (not case sensitive) 
                  
             DefaultButton (optional). Available options are: 
                 1, 2, 3 
  
             .OUTPUTS 
             Microsoft.VisualBasic.MsgBoxResult 
  
             .EXAMPLE 
             C:\PS> Show-MsgBox Hello 
             Shows a popup message with the text "Hello", and the default box, icon and defaultbutton settings. 
  
             .EXAMPLE 
             C:\PS> Show-MsgBox -Prompt "This is the prompt" -Title "This Is The Title" -Icon Critical -BoxType YesNo -DefaultButton 2 
             Shows a popup with the parameter as supplied. 
  
             .LINK 
             http://msdn.microsoft.com/en-us/library/microsoft.visualbasic.msgboxresult.aspx 
  
             .LINK 
             http://msdn.microsoft.com/en-us/library/microsoft.visualbasic.msgboxstyle.aspx 
             #> 
  
 function Show-MsgBox 
 { 
  
  [CmdletBinding()] 
     param( 
     [Parameter(Position=0, Mandatory=$true)] [string]$Prompt, 
     [Parameter(Position=1, Mandatory=$false)] [string]$Title ="", 
     [Parameter(Position=2, Mandatory=$false)] [ValidateSet("Information", "Question", "Critical", "Exclamation")] [string]$Icon ="Information", 
     [Parameter(Position=3, Mandatory=$false)] [ValidateSet("OKOnly", "OKCancel", "AbortRetryIgnore", "YesNoCancel", "YesNo", "RetryCancel")] [string]$BoxType ="OkOnly", 
     [Parameter(Position=4, Mandatory=$false)] [ValidateSet(1,2,3)] [int]$DefaultButton = 1 
     ) 
 [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.VisualBasic") | Out-Null 
 switch ($Icon) { 
             "Question" {$vb_icon = [microsoft.visualbasic.msgboxstyle]::Question } 
             "Critical" {$vb_icon = [microsoft.visualbasic.msgboxstyle]::Critical} 
             "Exclamation" {$vb_icon = [microsoft.visualbasic.msgboxstyle]::Exclamation} 
             "Information" {$vb_icon = [microsoft.visualbasic.msgboxstyle]::Information}} 
 switch ($BoxType) { 
             "OKOnly" {$vb_box = [microsoft.visualbasic.msgboxstyle]::OKOnly} 
             "OKCancel" {$vb_box = [microsoft.visualbasic.msgboxstyle]::OkCancel} 
             "AbortRetryIgnore" {$vb_box = [microsoft.visualbasic.msgboxstyle]::AbortRetryIgnore} 
             "YesNoCancel" {$vb_box = [microsoft.visualbasic.msgboxstyle]::YesNoCancel} 
             "YesNo" {$vb_box = [microsoft.visualbasic.msgboxstyle]::YesNo} 
             "RetryCancel" {$vb_box = [microsoft.visualbasic.msgboxstyle]::RetryCancel}} 
 switch ($Defaultbutton) { 
             1 {$vb_defaultbutton = [microsoft.visualbasic.msgboxstyle]::DefaultButton1} 
             2 {$vb_defaultbutton = [microsoft.visualbasic.msgboxstyle]::DefaultButton2} 
             3 {$vb_defaultbutton = [microsoft.visualbasic.msgboxstyle]::DefaultButton3}} 
 $popuptype = $vb_icon -bor $vb_box -bor $vb_defaultbutton 
 $ans = [Microsoft.VisualBasic.Interaction]::MsgBox($prompt,$popuptype,$title) 
 return $ans 
`

