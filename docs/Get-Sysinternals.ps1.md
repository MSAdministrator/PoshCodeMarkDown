---
Author: vbjay
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2682
Published Date: 2011-05-17t15
Archived Date: 2011-08-25t09
---

# get-sysinternals - 

## Description

downloads the current sysinternals from \\live.sysinternals.com\tools and then updates your path environment variable to include the location saved to.  along with that it sorts, and removes duplicate entries in the path variable.   if you call get-sysinternals without any args the local directory will be systemroot\sysinternals\  otherwise, the files will be put in the specified path.

## Comments



## Usage



## TODO



## function

`get-sysinternals`

## Code

`#
 #
  
   
   function Get-SysInternals {
 	
      param ( $sysIntDir=(join-path $env:systemroot "\Sysinternals\") )
 
    if(!(Test-Path -Path $sysIntDir -PathType Container)) 
     {
        $null = New-Item -Type Directory -Path $sysIntDir -Force 
     }
 	
    $log = join-path $sysIntDir "changes.log"
    Add-Content -force $log -value "`n`n[$(get-date)]SysInternals sync has been started"
   
   net use \\live.sysinternals.com\tools /persistent:no
   
   
    New-PSDrive -Name SYS -PSProvider filesystem -Root \\live.sysinternals.com\tools
 
    dir Sys: -recurse | foreach { 
 	
       $fileName = $_.name
       $localFile = join-path $sysIntDir $_.name                  
       $exist = test-path $localFile
 			
       $msgNew = "new utility found: $fileName , downloading..."
       $msgUpdate = "file : $fileName  is newer, updating..."
       $msgNoChange = "nothing changed for: $fileName"			
 	
       if($exist){
 	
             if($_.lastWriteTime -gt (Get-Item $localFile).lastWriteTime)
              {
                 Copy-Item $_.fullname $sysIntDir -force
                 Write-Host $msgUpdate -fore yellow
                 Add-Content -force $log -value $msgUpdate
              } 
             else 
              {
                 Add-Content $log -force -value $msgNoChange
                 Write-Host $msgNoChange
              }	
       }
       else
        {
             if($_.extension -eq ".exe")
              {
                   Write-Host $msgNew -fore green
                   Add-Content -force $log -value $msgNew
              } 
 	
 	   Copy-Item $_.fullname $sysIntDir -force 
       }
    }
    
   Update-Path "Sysinternals" $sysIntDir ""
   
     net use \\live.sysinternals.com\tools\ /delete
 }
 
 function Update-Path  ( [string] $Product, [string] $productPath, [string] $Property)
 
 {
 
 
 
 $pathArray = @()
 
 foreach ($pathItem in [Environment]::GetEnvironmentVariable('Path', 'Machine').Split(';')) {
     $pathItem = $pathItem.TrimEnd(('\'));
     $pathItem = $pathItem.Replace('"',"");
     #$pathItem =('"{0}"' -f $pathItem)
     if ($pathArray -contains $pathItem) {
         "Removing duplicate item: " + $pathItem;
     }
     else {
         "Keeping item: " + $pathItem;
         $pathArray += $pathItem
     }
 }
 
 
 "test path    "+$productPath
     if (Test-Path $productPath) {
         if ([string]::IsNullOrEmpty($Property)) {
         
         [string] $path = $productPath
         }
         else {
         [string] $path = (Get-ItemProperty $produuctPath).($Property)
         }
         #$path
         if (-not [string]::IsNullOrEmpty($path)) {
             $Product + " found"
              $path = $path.TrimEnd(('\'));
                  $path = $path.Replace('"',"");
                  #$path =('"{0}"' -f $path)
             if ($pathArray -notcontains $path ) {
                
                 "   Appending " + $path + " to path"
                 $pathArray += $path
             }
             else {
                 "    " + $path + "  already in path"
             }
         }
     }
     else {
         $product + " not found"
     }
 
 [Array]::Sort([array]$pathArray)
 
 #$pathArray
 
 
 
 ""
 "Old Path: " 
  ([Environment]::GetEnvironmentVariable('Path', 'Machine').Split(';'))|format-list
 ""
 ""
 [string] $newPath = [string]::Join(';', $pathArray);
 
 [Environment]::SetEnvironmentVariable('Path', $newPath, 'Machine');
 "New Path: " 
  ([Environment]::GetEnvironmentVariable('Path', 'Machine').Split(';'))|format-list
  
  
 }
 
 
  $wid=[System.Security.Principal.WindowsIdentity]::GetCurrent()
   $prp=new-object System.Security.Principal.WindowsPrincipal($wid)
   $adm=[System.Security.Principal.WindowsBuiltInRole]::Administrator
   $IsAdmin=$prp.IsInRole($adm)
   if($IsAdmin)
   {
 if ($args.Length -eq 0)
 {
 Get-Sysinternals
 }
 else
 {
 Get-Sysinternals $args[0]
 }
 }
 else
 {
 Write-Warning "This script requires running as an elevated administrator."
 }
`

