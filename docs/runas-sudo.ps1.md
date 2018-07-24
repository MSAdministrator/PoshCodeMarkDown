---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4534
Published Date: 2015-10-19t14
Archived Date: 2015-05-06t17
---

# runas\sudo - 

## Description

actually, start-process cmdlet with -verb runas can help you run process with admin’s rights, but… i do not like this cmdlet. i wanna something useable, so i wrote a prototype of something that looks like runas in cmd (or sudo in bash).

## Comments



## Usage



## TODO



## function

`invoke-runas`

## Code

`#
 #
 Set-Alias sudo Invoke-RunAs
 
 function Invoke-RunAs {
   <#
     .LINK
         Follow me on twitter @gregzakharov
         http://msdn.microsoft.com/en-US/library/system.diagnostics.aspx
   #>
   param(
     [Parameter(Mandatory=$true)]
     [String]$Program,
     
     [Parameter(Mandatory=$false)]
     [String]$Arguments,
     
     [Parameter(Mandatory=$false)]
     [Switch]$LoadProfile = $false,
     
     [Security.SecureString]$UserName = (Read-Host "Admin name" -as),
     [Security.SecureString]$Password = (Read-Host "Enter pass" -as)
   )
   
   function str([Security.SecureString]$s) {
     return [Runtime.InteropServices.Marshal]::PtrToStringAuto(
       [Runtime.InteropServices.Marshal]::SecureStringToBSTR($s)
     )
   }
   
   $psi = New-Object Diagnostics.ProcessStartInfo
   $psi.Arguments = $Arguments
   $psi.Domain = [Environment]::UserDomainName
   $psi.FileName = $Program
   $psi.LoadUserProfile = $LoadProfile
   $psi.Password = $Password
   $psi.UserName = (str $UserName)
   $psi.UseShellExecute = $false
   
   [void][Diagnostics.Process]::Start($psi)
 }
`

