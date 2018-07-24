---
Author: chris campbell
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3874
Published Date: 2013-01-10t19
Archived Date: 2013-01-22t13
---

# new-eicar - 

## Description

a powershell function to test that an antivirus product is working.

## Comments



## Usage



## TODO



## function

`new-eicar`

## Code

`#
 #
 function New-Eicar {
 <#
 .SYNOPSIS
  
     New-Eicar
        
     Author: Chris Campbell (@obscuresec)
     License: BSD 3-Clause
     
 .DESCRIPTION
 
     A function that generates the EICAR string to test ondemand scanning of antivirus products.
 
 .PARAMETER $Path
 
     Specifies the path to write the eicar file to.
 
 .EXAMPLE
 
     PS C:\> New-Eicar -Path c:\test 
 
 .NOTES
 
     During testing, several AV products caused the script to hang, but it always completed after a few minutes.
 
 .LINK
 
     http://obscuresec.com/2013/01/New-Eicar.html
     https://github.com/obscuresec/random/blob/master/New-Eicar
     
 #>
     [CmdletBinding()] Param(
         [ValidateScript({Test-Path $_ -PathType 'Container'})] 
         [string] 
         $Path = "$env:temp\"
         )            
             [string] $FilePath = (Join-Path $Path eicar.com)
             [string] $EncodedEicar = 'WDVPIVAlQEFQWzRcUFpYNTQoUF4pN0NDKTd9JEVJQ0FSLVNUQU5EQVJELUFOVElWSVJVUy1URVNULUZJTEUhJEgrSCo='
 
             If (!(Test-Path -Path $FilePath)) {
 
                 Try {
                     [byte[]] $EicarBytes = [System.Convert]::FromBase64String($EncodedEicar)
                     [string] $Eicar = [System.Text.Encoding]::UTF8.GetString($EicarBytes)
                     Set-Content -Value $Eicar -Encoding ascii -Path $FilePath -Force 
                 }
 
                 Catch {
                     Write-Warning "Eicar.com file couldn't be created. Either permissions or AV prevented file creation."
                 }
             }
             
             Else {
                 Write-Warning "Eicar.com already exists!"
             }
 
 }
`

