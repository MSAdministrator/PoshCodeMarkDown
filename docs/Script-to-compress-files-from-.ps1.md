---
Author: shadow
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 411
Published Date: 
Archived Date: 2012-06-30t05
---

#  - zip.ps1

## Description

script to compress files from command line.

## Comments



## Usage



## TODO



## 

``

## Code

`#
 param([string]$zipfilename=$(throw "ZIP archive name needed!"), 
       [string[]]$filetozip=$(throw "Supply the file(s) to compress (wildcards accepted)"))
 trap [Exception] {break;}
 
 $emptyzipfile=80,75,5,6,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0;
 
 $fmode = [System.IO.FileMode]::Create;
 $zipfile = new-object -type System.IO.FileStream $zipfilename, $fmode;
 $zipfile.Write($emptyzipfile,0,22);
 $zipfile.Flush();
 $zipfile.Close();
 
 $shell=new-object -comobject Shell.Application;
 $foldername=(ls $zipfile.Name).FullName;
 $zipfolder=$shell.Namespace($foldername);
 
 Write-Host "Compressing:";
 $filetozip| %{ls $_}| 
     foreach{ 
         ("`t"+$_.fullname);
         $zipfolder.copyhere($_.fullname,20);
         [System.Threading.Thread]::Sleep(250);
     }
 $shell=$null;
 $zipfolder=$null;
`

