---
Author: battlechicken
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5807
Published Date: 2015-04-01t18
Archived Date: 2015-04-03t14
---

# secure passwords - 

## Description

encrypting and decrypting passwords using securestring.  simple example.

## Comments



## Usage



## TODO



## function

`get-passwordfromencryptedfile`

## Code

`#
 #
 Function Get-PasswordFromEncryptedFile {
 <#
 .Synopsis
    Converts a password stored as a secure string to a file to plain text.
 .DESCRIPTION
    Converts a password stored as a secure string to a file to plain text.
 .EXAMPLE
    Get-PasswordFromEncryptedFile -PasswordFile "c:\admin\MyEncryptedPass.txt"
    Assuming the user who encryptedt he password is the same user executing the command, will decrypt the string in c:\admin\MyEncryptedPass.txt to plain-text.
 .OUTPUTS
    Outputs a string object
 .NOTES
    This function can be tricky.  it decrypts a securestring, so it will only be reversible by the same user that created the original encrypted file.  So, if my user is MyDomain\MyUsername, only MyDomain\MyUsername on the same machine can reverse the encryption.  Keep in mind the decrypt will only work if you created the file on that same machine.
 .FUNCTIONALITY
    Decrypts a secure string saved to a file.
 #>
     param(
         [parameter(Mandatory=$true)]
         [string]$PasswordFile
     )
 
     if (-not (Test-Path $PasswordFile)){
         throw "Nonexistent Password file"
     }
     else {
         try{
             $encryptedPass = get-content $PasswordFile | ConvertTo-SecureString
             $encryptedStr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($encryptedPass)
             [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($encryptedStr)
         }
         catch {
             throw "Error decrypting Secure string.  Only files encrypted by $env:USERNAME on $env:COMPUTERNAME can be decrypted in this session."
         }
 
     }
 }
 
 Function New-PasswordFile {
 <#
 .Synopsis
    Saves a string (a password most likely) to the specified file.
 .DESCRIPTION
    Saves a string (a password most likely) to the specified file.
 .EXAMPLE
    New-PasswordFile -PasswordFile c:\admin\MyEncryptedPassword.txt
    Prompts the user for a string, which gets saved encrypted to c:\admin\MyEncryptedPassword.txt
 #>
     param(
         [parameter(Mandatory=$true)]
         [string]$PasswordFile
     )
 
     read-host -AsSecureString "Enter a password" | ConvertFrom-SecureString -ErrorAction stop| out-file $PasswordFile -ErrorAction Stop
`

