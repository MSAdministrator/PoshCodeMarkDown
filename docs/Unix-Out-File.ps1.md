---
Author: bruno martins
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6308
Published Date: 2016-04-18t17
Archived Date: 2016-04-22t21
---

# unix out-file - 

## Description

write line to file with unix line endings

## Comments



## Usage



## TODO



## function

`out-unixfile`

## Code

`#
 #
 function Out-UnixFile
 {
     Param(
 
         [Parameter( Mandatory = $True )]
         [string]
         $FilePath,
 
         [Parameter( Mandatory = $True, ValueFromPipeline = $True )]
         [string[]]
         $InputObject,
 
         [switch]
         $Append = $False
     )
 
     BEGIN
     {
         $Fs = $Null
         $Sw = $Null
         
         Try
         {
             If( -not( Test-Path $FilePath ) )
             {
                 $result = New-Item -Path $FilePath -ItemType File -ErrorAction Stop
             }
 
             $Encoding = New-Object System.Text.UTF8Encoding
 
             If( $Append )
             {
                 $FileMode = [System.IO.FileMode]::Append
             }
             Else
             {
                 $FileMode = [System.IO.FileMode]::Open
             }
 
             $Fs = New-Object -TypeName System.IO.FileStream( $FilePath, $FileMode )
             $Sw = New-Object -TypeName System.IO.StreamWriter( $Fs, $Encoding )
         }
         Catch
         {
             Throw
         }
     }
 
     PROCESS
     {
 
         Try
         {
             ForEach( $Line in $InputObject )
             {
                 $Sw.Write( "$( $line )`n" )
             }
         }
         Catch
         {
             If( $Sw -ne $Null )
             {
                 $Sw.Close()
             }
 
             If( $Fs -ne $Null )
             {
                 $Fs.Close()
             }
 
             Throw
         }
     }
 
     END
     {
         $Sw.Close()
         $Fs.Close()
     }
 }
`

