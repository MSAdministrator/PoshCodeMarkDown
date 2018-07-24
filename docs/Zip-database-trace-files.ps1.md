---
Author: kevin bullen
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3672
Published Date: 2012-10-01t12
Archived Date: 2012-10-21t16
---

# zip database trace files - 

## Description

i wrote this scrip to find and zip database trace files.  there are switch parameters to search subdirectories recursively as well as removing the original trace file.

## Comments



## Usage



## TODO



## function

`zip-tracefiles`

## Code

`#
 #
 Clear-Host
 
 function Zip-TraceFiles { 
     Param(
           [string]$DirToSearch,
           [switch]$SearchSubDirs,
           [switch]$RemoveTraceFile
           ) 
     
     If ($SearchSubDirs -eq -$true) {
         $TraceFiles = Get-ChildItem -Path $DirToSearch | Where-Object {$_.Extension -eq '.trc'};
     }
     else {
         $TraceFiles = Get-ChildItem -Path $DirToSearch -Recurse | Where-Object {$_.Extension -eq '.trc'};    
     }    
         
     If ($TraceFiles -ne $null) {
         ForEach($TraceFile in $TraceFiles)
         {                
             $FullTraceFile = $($TraceFile.Directory.ToString() + "\" + $TraceFile.Name);
             
             Write-Host "Zipping File $FullTraceFile";
             $NewZipFile = $FullTraceFile;              
 
             If (-not $NewZipFile.EndsWith('.zip')) {
                 $NewZipFile += '.zip';
             } 
 
             If (Test-Path -path $NewZipFile -PathType Leaf) {
                 Remove-Item -path $NewZipFile -Force;
             }
 
             Set-Content $NewZipFile ("PK" + [char]5 + [char]6 + ("$([char]0)" * 18));
 
             try { 
                 $ZipFile = (New-Object -com shell.application).NameSpace($NewZipFile);
          
                 $FileToZip =  Get-Item -path $FullTraceFile
          
                 $ZipFile.CopyHere( $FileToZip.fullname );
                 $size = $ZipFile.Items().Item($FileToZip.Name).Size;
             
                 While($ZipFile.Items().Item($FileToZip.Name) -eq $null)
                 {
                     Start-Sleep -Seconds 1;
                     Write-Host "." -NoNewline;
                 }
             
                 Write-Host ".";
                  
                 If ($RemoveTraceFile -eq $true) {
                     If (Test-Path -path $FullTraceFile -PathType Leaf) {
                         Remove-Item -path $FullTraceFile -Force;
                     }
                 }        
             }
             catch {   
                 Write-Host $_.Exception.ToString()    
 
 }  
 
 
 Zip-TraceFiles -DirToSearch 'C:\TEMP\_psdev' -SearchSubDirs -RemoveTraceFile
`

