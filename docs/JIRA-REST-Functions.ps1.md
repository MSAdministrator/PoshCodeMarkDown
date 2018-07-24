---
Author: chris916
Publisher: 
Copyright: 
Email: 
Version: 5.1.7
Encoding: ascii
License: cc0
PoshCode ID: 5471
Published Date: 2016-09-29t20
Archived Date: 2016-11-30t10
---

# jira rest functions - 

## Description

powershell jira rest method authentication and common functions…

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 $jiraURL = "jira.companyname.com"
 
 function global:get-JiraAttach ($url, $file){
         Invoke-WebRequest -uri "https://$jiraURL/login.jsp?os_username=$jLogin&os_password=$jPassword&os_cookie=true" -UserAgent ([Microsoft.PowerShell.Commands.PSUserAgent]::InternetExplorer) -SessionVariable myWebSession
         Invoke-WebRequest -uri $url -OutFile $file -method get -UserAgent ([Microsoft.PowerShell.Commands.PSUserAgent]::InternetExplorer) -WebSession $myWebSession
 }
 
 function global:get-ParentIssueNum ($issueNum){
         $urlREST = "https://$jiraURL/rest/api/latest/search?maxResults=100&fields=key&jql=issueFunction%20in%20parentsOf(%27Key=$issueNum%27)"
         $ret = Invoke-WebRequest -uri "https://$jiraURL/login.jsp?os_username=$jLogin&os_password=$jPassword&os_cookie=true" -UserAgent ([Microsoft.PowerShell.Commands.PSUserAgent]::InternetExplorer) -SessionVariable myWebSession
         $jsonStr = Invoke-WebRequest -uri $urlREST -method get -UserAgent ([Microsoft.PowerShell.Commands.PSUserAgent]::InternetExplorer) -WebSession $myWebSession
         $parentKey = MID $jsonStr.Content "`"key`":`"" "`"}]}"
         return $parentKey
 }
 
 function global:set-JiraAssign ($issueKey, $assignName){
     #$assignName can be the Jira login name
     $body = '{ "name": "'+$assignName+'" }'
     $restapiuri = "https://$jiraURL/rest/api/latest/issue/"+$issueKey+"/assignee"
     $jsonresponse = Invoke-RestMethod -Uri $restapiuri -Headers @{"Authorization"="Basic $encodedCredentials"} -ContentType application/json -method put -body $body
 }
 
 function global:set-JiraIssueStatus ($issueKey, $status){
     if ($status.ToUpper() -eq "START PROGRESS"){
         $body = '{ "transition" : { "id" : "11" } }'
     }
     elseif ($status.ToUpper() -eq "PLACE ON HOLD"){
         $body = '{ "transition" : { "id" : "31" } }'
     }
     elseif ($status.ToUpper() -eq "CLOSE"){
         $body = '{ "transition" : { "id" : "51" } }'
     }
     else{
         write-host "Not a scripted status to transition to!"
         sleep-start 2
         exit
     }
 
     #$restapiuri = "https://$jiraURL/rest/api/latest/issue/"+$issueKey+"/transitions"
     #$jsonresponse = Invoke-RestMethod -Uri $restapiuri -Headers @{"Authorization"="Basic $encodedCredentials"} -ContentType application/json -method get
     #$jsonresponse | out-file "C:\json_transitions_out.txt"
 
     $restapiuri = "https://$jiraURL/rest/api/latest/issue/"+$issueKey+"/transitions"
     $jsonresponse = Invoke-RestMethod -Uri $restapiuri -Headers @{"Authorization"="Basic $encodedCredentials"} -ContentType application/json -method post -body $body
 }
 
 Function MID($mainStr, $firstStr, $secondStr) {
     $a=$mainStr
     $b = $a.indexof($firstStr)
     $b = $b + $firstStr.length
     $c = $a.indexof($secondStr)
     $d = $c - $b
     $e = $a.substring($b,$d)
     return $e
 }
 
 if (!$credential){
     $global:credential = get-credential -Message "Please enter Jira credentials:"
 
     $BSTR = [System.Runtime.InteropServices.marshal]::SecureStringToBSTR($credential.Password)
     $global:jPassword = [System.Runtime.InteropServices.marshal]::PtrToStringAuto($BSTR)
     $global:jLogin = $credential.UserName.TrimStart("\")
 
     $bytes = [System.Text.Encoding]::UTF8.GetBytes("$jLogin`:$jPassword")
     $global:encodedCredentials = [System.Convert]::ToBase64String($bytes)
 }
`

