---
Author: ian philpot
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6127
Published Date: 2016-12-02t02
Archived Date: 2016-03-19t02
---

# scrum labels in github - 

## Description

use this script to create scrum issue labels for a github repo.

## Comments



## Usage



## TODO



## function

`delete-label`

## Code

`#
 #
 param([string]$OwnerName = (Read-Host "What is the owner name?"),
       [string]$RepositoryName = (Read-Host "What is the repository name?"),
       [string]$AuthToken = (Read-Host "What is the auth token?"),
       [switch]$DeleteLabels)
 
 $labelJson = @"
 [
     {
         "name":  "priority:lowest",
         "color":  "207de5"
     },
     {
         "name":  "priority:low",
         "color":  "207de5"
     },
     {
         "name":  "priority:medium",
         "color":  "207de5"
     },
     {
         "name":  "priority:high",
         "color":  "207de5"
     },
     {
         "name":  "priority:highest",
         "color":  "207de5"
     },
     {
         "name":  "point:1",
         "color":  "009800"
     },
     {
         "name":  "point:2",
         "color":  "009800"
     },
     {
         "name":  "point:3",
         "color":  "009800"
     },
     {
         "name":  "point:5",
         "color":  "009800"
     },
     {
         "name":  "point:8",
         "color":  "009800"
     },
     {
         "name":  "point:13",
         "color":  "009800"
     },
     {
         "name":  "type:bug",
         "color":  "eb6420"
     },
     {
         "name":  "type:chore",
         "color":  "eb6420"
     },
     {
         "name":  "type:feature",
         "color":  "eb6420"
     },
     {
         "name":  "type:infrastructure",
         "color":  "eb6420"
     },
     {
         "name":  "type:performance",
         "color":  "eb6420"
     },
     {
         "name":  "type:refactor",
         "color":  "eb6420"
     },
     {
         "name":  "type:test",
         "color":  "eb6420"
     }
 ]
 
 "@
 
 $headers = @{"Authorization"="token $AuthToken"}
 $labelList = $labelJson | ConvertFrom-Json
 
 function Delete-Label {
     param([string]$lableName)
 
     $url = "https://api.github.com/repos/{0}/{1}/labels/{2}" -f $OwnerName, $RepositoryName, $lableName
 
     Invoke-WebRequest $url -Method Delete -Headers $headers
 }
 
 function Create-Label {
     param([string]$lableName, [string]$labelColor)
 
     $hashTable = @{"name"=$lableName; "color"=$labelColor}
     $data = $hashTable | ConvertTo-Json
 
     $url = "https://api.github.com/repos/{0}/{1}/labels" -f $OwnerName, $RepositoryName
 
     Invoke-WebRequest $url -Method Post -Body $data -Headers $headers
 }
 
 function Get-CurrentLabels {
 	$url = "https://api.github.com/repos/{0}/{1}/labels" -f $OwnerName, $RepositoryName
     $result = (Invoke-WebRequest $url -Headers $headers).Content
 	$labels = $result | ConvertFrom-Json
 
 	return $labels
 }
 
 if ($DeleteLabels) {
 	$labelList = Get-CurrentLabels
 
     foreach ($label in $labelList) {
         Write-Host "Deleting Label:" $label.name -f Yellow
         $result = Delete-Label -lableName $label.name
 
         if ($result.StatusCode -eq 204) {
             Write-Host $label.name "was deleted" -f DarkYellow
         } else {
             Write-Host $label.name "was not deleted" -f DarkRed
         }
     }
 }
 
 if (!$DeleteLabels) {
     foreach ($label in $labelList) {
         Write-Host "Creating Label:" $label.name -f Yellow
         $result = Create-Label -lableName $label.name -labelColor $label.color
 
         if ($result.StatusCode -eq 201) {
             Write-Host $label.name "was created" -f DarkYellow
         } else {
             Write-Host $label.name "was not created" -f DarkRed
         }
         
     }
 }
`

