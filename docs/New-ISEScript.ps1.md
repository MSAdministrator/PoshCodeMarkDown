---
Author: biryani
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 4477
Published Date: 2013-09-18t10
Archived Date: 2013-10-05t04
---

# new-isescript.ps1 - 

## Description

just a quick re-write of gmagerr’s powershell template script to enable his template to create powershell ise help-comment based structure which i often use; i just put this function into an add-isemenu custom menu array. thanks to gmagerr for the structure i just added a couple of things (most of which work fine!). thx for the base gmagerr.

## Comments



## Usage



## TODO



## function

`new-isescript`

## Code

`#
 #
 Function New-ISEScript
 {
 $strName = $env:username
 $date = get-date -format d
 $name = Read-Host "Filename"
 if ($name -eq "") { $name="NewScriptTemplate_ISECommentBased" }
 $synopsis = Read-Host "Synopsis"
 if ($synopsis -eq "") { $synopsis="enter your synopsis of this script activity here" }
 $description = Read-Host "Description"
 if ($description -eq "") { $description="enter a brief description of this script here" }
 $syntax = Read-Host "Syntax"
 if ($syntax -eq "") { $syntax="enter syntax here" }
 $email = Read-Host "eMail Address"
 if ($email -eq "") { $email='myemail@mycompanyemail.com' }
 $example1 = Read-Host "Example 1"
 if ($example1 -eq "") { $example1="enter example 1 here" }
 $example2 = Read-Host "Example 2"
 if ($example2 -eq "") { $example2="enter example 2 here" }
 $helpurl1 = Read-Host "enter the help url 1 here"
 if ($helpurl1 -eq "") { $helpurl1="enter help url 1 here" }
 $helpurl2 = Read-Host "enter the help url 2 here"
 if ($helpurl2 -eq "") { $helpurl2="enter help url 2 here" }
 $seealso1 = Read-Host "See Also 1"
 if ($seealso1 -eq "") { $seealso1="enter see also 1 here" }
 $seealso2 = Read-Host "See Also 2"
 if ($seealso2 -eq "") { $seealso2="enter see also 2 here" }
 $seealso3 = Read-Host "See Also 3"
 if ($seealso3 -eq "") { $seealso3="enter see also 3 here" }
 $comment=@();
 while($s = (Read-Host "Comment").Trim()){$comment+="$s`r`n#"}
 $file = New-Item -type file "$name.ps1" -force
 
 $template=@"
 <#
  .NAME
  	$name.ps1
  .SYNOPSIS
  	$synopsis
  .DESCRIPTION
  	$description
  .SYNTAX
  	$syntax
  .EXAMPLES
  	$example1
 	$example2
  .NOTES
  	AUTHOR:	$strName
   	EMAIL:	$email
 	DATE:	$date
   	COMMENT: $comment
  .HELPURL
  	$helpurl1
 	$helpurl2
  .SEEALSO
  	$seealso1
 	$seealso2
 	$seealso3
 #>
 
 ###########################################################################
 #
 #
 #
 ###########################################################################
 
 
 
 "@ | out-file $file
 ii $file
 }
  
 Set-Alias new New-ISEScript
`

