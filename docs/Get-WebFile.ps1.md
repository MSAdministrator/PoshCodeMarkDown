---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 3.6
Encoding: ascii
License: cc0
PoshCode ID: 4331
Published Date: 2014-07-25t02
Archived Date: 2014-01-14t02
---

# get-webfile - 

## Description

an upgrade to my wget script which can output the downloaded html to the pipeline. get-webfile can download text or binary files, automatically determine file names, and present a progress report for large files…

## Comments



## Usage



## TODO



## script

`get-webfile`

## Code

`#
 #
 ##############################################################################################################
 ##############################################################################################################
 function Get-WebFile {
    param( 
       $url = (Read-Host "The URL to download"),
       $fileName = $null,
       [switch]$Passthru,
       [switch]$quiet
    )
    
    $req = [System.Net.HttpWebRequest]::Create($url);
    $res = $req.GetResponse();
  
    if($fileName -and !(Split-Path $fileName)) {
       $fileName = Join-Path (Get-Location -PSProvider "FileSystem") $fileName
    } 
    elseif((!$Passthru -and ($fileName -eq $null)) -or (($fileName -ne $null) -and (Test-Path -PathType "Container" $fileName)))
    {
       [string]$fileName = ([regex]'(?i)filename=(.*)$').Match( $res.Headers["Content-Disposition"] ).Groups[1].Value
       $fileName = $fileName.trim("\/""'")
       if(!$fileName) {
          $fileName = $res.ResponseUri.Segments[-1]
          $fileName = $fileName.trim("\/")
          if(!$fileName) { 
             $fileName = Read-Host "Please provide a file name"
          }
          $fileName = $fileName.trim("\/")
          if(!([IO.FileInfo]$fileName).Extension) {
             $fileName = $fileName + "." + $res.ContentType.Split(";")[0].Split("/")[1]
          }
       }
       $fileName = Join-Path (Get-Location -PSProvider "FileSystem") $fileName
    }
    if($Passthru) {
       $encoding = [System.Text.Encoding]::GetEncoding( $res.CharacterSet )
       [string]$output = ""
    }
  
    if($res.StatusCode -eq 200) {
       [int]$goal = $res.ContentLength
       $reader = $res.GetResponseStream()
       if($fileName) {
          $writer = new-object System.IO.FileStream $fileName, "Create"
       }
       [byte[]]$buffer = new-object byte[] 4096
       [int]$total = [int]$count = 0
       do
       {
          $count = $reader.Read($buffer, 0, $buffer.Length);
          if($fileName) {
             $writer.Write($buffer, 0, $count);
          } 
          if($Passthru){
             $output += $encoding.GetString($buffer,0,$count)
          } elseif(!$quiet) {
             $total += $count
             if($goal -gt 0) {
                Write-Progress "Downloading $url" "Saving $total of $goal" -id 0 -percentComplete (($total/$goal)*100)
             } else {
                Write-Progress "Downloading $url" "Saving $total bytes..." -id 0
             }
          }
       } while ($count -gt 0)
       
       $reader.Close()
       if($fileName) {
          $writer.Flush()
          $writer.Close()
       }
       if($Passthru){
          $output
       }
    }
    $res.Close(); 
    if($fileName) {
       ls $fileName
    }
 }
`

