---
Author: x0wllaar
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5696
Published Date: 2015-01-19t14
Archived Date: 2015-01-31t20
---

# "password manager" - 

## Description

got bored, decided to create this.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $alphabet = '!"¹;%:?*()_+=-~/\<>,.[]{}1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz';$iter=343
 [Console]::Write("Enter your master password: ")
 $pass="";while($true){$key=[Console]::ReadKey($true);if($key.Key-eq[ConsoleKey]::Backspace){[Console]::Write("`b `b");$pass = $pass.Substring(0, ($pass.Length - 1))}elseif($key.Key-eq[ConsoleKey]::Enter){break}else{$pass+=$key.KeyChar;[Console]::Write("*")}};""
 $htool=New-Object System.Security.Cryptography.SHA512Managed
 $hpass=$htool.ComputeHash([System.Text.Encoding]::UTF8.GetBytes($pass))
 $creds=@()
 (Import-Csv $args[0])|%{
     $hsite = $htool.ComputeHash([System.Text.Encoding]::UTF8.GetBytes($_.SiteName)); $hname = $htool.ComputeHash([System.Text.Encoding]::UTF8.GetBytes($_.AccName))
     $interw=@(); (0..63)|%{$interw+=$hsite[$_];$interw+=$hpass[$_];$interw+=$hname[$_];}
     $seed=$htool.ComputeHash($interw)
     (0..($iter+$_.PassNum-1))|%{$seed=$htool.ComputeHash($seed)}
     $spass="";(0..($_.Len-1))|%{$spass += $alphabet[($seed[$_] % $alphabet.Length)]}
     $cred = New-Object -TypeName PSObject; 
     Add-Member -InputObject $cred -MemberType NoteProperty -Name SiteName -Value $_.SiteName; 
     Add-Member -InputObject $cred -MemberType NoteProperty -Name AccName -Value $_.AccName; 
     Add-Member -InputObject $cred -MemberType NoteProperty -Name Pass -Value $spass
     $creds += $cred
 }
 $creds
`

