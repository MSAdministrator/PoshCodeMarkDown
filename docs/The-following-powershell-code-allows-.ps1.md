---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5639
Published Date: 
Archived Date: 2015-02-17t21
---

#  - 

## Description

the following powershell code allows you to log into jira, create an issue, assign the issue to someone (yourself in this case), add two comments, resolve, and close.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 $jirasvc = New-WebServiceProxy -Uri "http://jra.netpost/jira//rpc/soap/jirasoapservice-v2?wsdl"
 $username = "myuserid"
 $password = "mypassword"
 
 $token = $jirasvc.login($username,$password)
 
 $namespace = $jirasvc.GetType().Namespace
 
 $issue = New-Object "$namespace.RemoteIssue"
 
 $Jira_Component = New-Object "$namespace.RemoteComponent"
 $Jira_Component.id = 12740
 $Jira_Component.name = "SCM" 
  
 $issue.summary = "a summary" 
 $issue.description = "foo mane padme hum" 
 $issue.components += $Jira_Component 
 $issue.project = "TEST"
 
 $remoteIssue = $jirasvc.createIssue($token,$issue)
 
 Write-Host $remoteIssue.key
 
 $remoteField = New-Object "$namespace.RemoteFieldValue"
 $remoteField.id = "assignee"
 $remoteField.values = @( "laenenj" ) 
 
 $remoteIssue = $jirasvc.updateIssue($token,$remoteIssue.key, $remoteField)
 
 $startWorking = New-Object "$namespace.RemoteFieldValue"
 $startWorking.id = "Start working"
  
 $remoteIssue = $jirasvc.progressWorkflowAction($token,$remoteIssue.key,"751",$startWorking)
 
 $remoteComment = New-Object "$namespace.RemoteComment"
 $remoteComment.body = "A first comment"
 $jirasvc.addComment($token,$remoteIssue.key,$remoteComment)
 
 $remoteComment.body = "A second comment"
 $jirasvc.addComment($token,$remoteIssue.key,$remoteComment)
 
 $resolve = New-Object "$namespace.RemoteFieldValue"
 $resolve.id = "Finish"
  
 $remoteIssue = $jirasvc.progressWorkflowAction($token,$remoteIssue.key,"5",$resolve)
 
 $close = New-Object "$namespace.RemoteFieldValue"
 $close.id = "Close"
  
 $remoteIssue = $jirasvc.progressWorkflowAction($token,$remoteIssue.key,"2",$close)
`

