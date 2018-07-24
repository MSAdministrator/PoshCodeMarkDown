---
Author: jaydeuce
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6235
Published Date: 2016-02-25t05
Archived Date: 2016-03-26t12
---

# get-gitinfofordirectory - 

## Description

get-gitinfofordirectory function – git repo folder info gather function to display git repo status and branch in the powershell prompt. – add function to your profile script, and call from within the prompt function – must have git for windows installed and its install directory added to the path

## Comments



## Usage



## TODO



## function

`get-gitinfofordirectory`

## Code

`#
 #
 function Get-GitInfoForDirectory {
 
     param (
     )
 
     begin {
         $gitBranch = (git branch)
         $gitStatus = (git status)
         $gitTextLine = ""
     }
 
     process {
         try {
             foreach ($branch in $gitBranch) {
                 if ($branch -match '^\* (.*)') {
                     $gitBranchName = 'Git Repo - Branch: ' + $matches[1].ToUpper()
     	        }
             }
     
             if (!($gitStatus -like '*working directory clean*')) {
                 $gitStatusMark = ' ' + '/' + ' Status: ' + 'NEEDS UPDATING'
             }
             elseif ($gitStatus -like '*Your branch is ahead*') {
                 $gitStatusMark = ' ' + '/' + ' Status: ' + 'PUBLISH COMMITS'
             }
             else {
                 $gitStatusMark = ' ' + '/' + ' Status: ' + 'UP TO DATE'
             }
         }
         catch {
         }
     }
 
     end {
         if ($gitBranch) { 
             $gitTextLine = ' {' + $gitBranchName + $gitStatusMark + '}'            
         }
         return $gitTextLine       
     }    
 }
`

