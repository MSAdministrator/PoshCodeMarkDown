---
Author: robbie foust
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 751
Published Date: 2009-12-26t20
Archived Date: 2015-12-09t19
---

# atlassian jira interface - 

## Description

this is a set of powershell functions to interface with the atlassian jira bug/issue tracking software using a wsdl interface.

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 #
 #
 #
 #
 #
 #
 
 $global:jiraURL = "https://server.yourdomain.com/jira/rpc/soap/jirasoapservice-v2?wsdl"
 
 function global:connect-jira ($wsdlLocation)
 {
 	##############################################################################
 	##
 	##
 	##
 	##
 	##
 	##############################################################################
 
 	if(-not (Test-Path Variable:\Lee.Holmes.WebServiceCache))
 	{
 	    ${GLOBAL:Lee.Holmes.WebServiceCache} = @{}
 	}
 
 	$oldInstance = ${GLOBAL:Lee.Holmes.WebServiceCache}[$wsdlLocation]
 	if($oldInstance)
 	{
 	    $oldInstance
 	    return
 	}
 
 	[void] [Reflection.Assembly]::LoadWithPartialName("System.Web.Services")
 
 	$wc = new-object System.Net.WebClient
 
 	if($requiresAuthentication)
 	{
 	    $wc.UseDefaultCredentials = $true
 	}
 
 	$wsdlStream = $wc.OpenRead($wsdlLocation)
 
 	if(-not (Test-Path Variable:\wsdlStream))
 	{
 	    return
 	}
 
 	$serviceDescription =
 	    [Web.Services.Description.ServiceDescription]::Read($wsdlStream)
 	$wsdlStream.Close()
 
 	if(-not (Test-Path Variable:\serviceDescription))
 	{
 	    return
 	}
 
 	$serviceNamespace = New-Object System.CodeDom.CodeNamespace
 	if($namespace)
 	{
 	    $serviceNamespace.Name = $namespace
 	}
 
 	$codeCompileUnit = New-Object System.CodeDom.CodeCompileUnit
 	$serviceDescriptionImporter = 
 	    New-Object Web.Services.Description.ServiceDescriptionImporter
 	$serviceDescriptionImporter.AddServiceDescription(
 	    $serviceDescription, $null, $null)
 	[void] $codeCompileUnit.Namespaces.Add($serviceNamespace)
 	[void] $serviceDescriptionImporter.Import(
 	    $serviceNamespace, $codeCompileUnit)
 
 	$generatedCode = New-Object Text.StringBuilder
 	$stringWriter = New-Object IO.StringWriter $generatedCode
 	$provider = New-Object Microsoft.CSharp.CSharpCodeProvider 
 	$provider.GenerateCodeFromCompileUnit($codeCompileUnit, $stringWriter, $null)
 
 	$references = @("System.dll", "System.Web.Services.dll", "System.Xml.dll")
 	$compilerParameters = New-Object System.CodeDom.Compiler.CompilerParameters 
 	$compilerParameters.ReferencedAssemblies.AddRange($references)
 	$compilerParameters.GenerateInMemory = $true
 
 	$compilerResults = 
 	    $provider.CompileAssemblyFromSource($compilerParameters, $generatedCode)
 
 	if($compilerResults.Errors.Count -gt 0) 
 	{ 
 	    $errorLines = "" 
 	    foreach($error in $compilerResults.Errors) 
 	    { 
 		$errorLines += "`n`t" + $error.Line + ":`t" + $error.ErrorText 
 	    } 
 
 	    Write-Error $errorLines
 	    return 
 	}
 	else 
 	{
 	    $assembly = $compilerResults.CompiledAssembly
 
 	    $type = $assembly.GetTypes() |
 		Where-Object { $_.GetCustomAttributes(
 		    [System.Web.Services.WebServiceBindingAttribute], $false) }
 
 	    if(-not $type)
 	    {
 		Write-Error "Could not generate web service proxy."
 		return
 	    }
 
 	    $instance = $assembly.CreateInstance($type)
 
 	    if($requiresAuthentication)
 	    {
 		if(@($instance.PsObject.Properties | 
 		    where { $_.Name -eq "UseDefaultCredentials" }).Count -eq 1)
 		{
 		    $instance.UseDefaultCredentials = $true
 		}
 	    }
 	    
 	    ${GLOBAL:Lee.Holmes.WebServiceCache}[$wsdlLocation] = $instance
 
 	    $instance
 	}
 }
 
 
 function global:get-JiraServerInfo
 	{
 	$jira.GetServerInfo($jiraAuthID)
 	}
 
 
 function global:get-JiraIssueType
 	{
 	$jira.GetIssueTypes($jiraAuthID)
 	}
 
 function global:get-JiraSubtaskIssueType
 	{
 	$jira.GetSubtaskIssueTypes($jiraAuthID)
 	}
 
 function global:get-JiraStatus
 	{
 	$jira.GetStatuses($jiraAuthID)
 	}
 
 function global:get-JiraPriority
 	{
 	$jira.GetPriorities($jiraAuthID)
 	}
 
 function global:get-JiraResolution
 	{
 	$jira.GetResolutions($jiraAuthID)
 	}
 
 function global:get-JiraReport
 	{
 	$jira.GetSavedFilters($jiraAuthID)
 	}
 
 function global:get-JiraProject
 	{
 	$jira.GetProjects($jiraAuthID)
 	}
 
 function global:get-JiraComment ($issueKey)
 	{
 	$jira.GetComments($jiraAuthID,$issueKey)
 	}
 
 function global:new-JiraComment ($issueKey, $comment)
 	{
 	$jiraComment = new-object RemoteComment
 	$jiraComment.body = $comment
 
 	$jira.AddComment($jiraAuthID, $issueKey, $jiraComment)
 	}
 
 function global:export-JiraReport ($reportNumber)
 	{
 	$jira.GetIssuesFromFilter($jiraAuthID, $reportNumber)
 	}
 
 function global:update-JiraIssue ([string]$issueKey)
 	{
 	
 	$jira.UpdateIssue($jiraAuthID,$issueKey,$placeholder)
 	} 
 
 function global:set-JiraIssueStatus ($issueKey,$actionID,$placeholder)
 	{
 	$jira.ProgressWorkflowAction($jiraAuthID,$issueKey,$actionID,$placeholder)
 	}
 
 function global:get-JiraIssue ($issueKey)
 	{
 	$jira.GetIssue($jiraAuthID, $issueKey)
 	}
 
 function global:new-JiraIssue ($project, $type, $summary, $description)
 	{
 	$jiraIssue = new-object RemoteIssue
 	$jiraIssue.project = $project
 	$jiraIssue.type = $type
 	$jiraIssue.summary = $summary
 	$jiraIssue.description = $description
 
 	$newIssue = $jira.CreateIssue($jiraAuthID, $jiraIssue)
 	
 	$newIssue
 	}
 
 function global:disconnect-jira
 	{
 	$jira.logout($jiraAuthID)
 	}
 
 
 $global:jira = connect-jira $jiraURL
 
 if (!$credential)
 	{
 	$global:credential = get-credential
 	}
 
 $BSTR = [System.Runtime.InteropServices.marshal]::SecureStringToBSTR($credential.Password)
 $global:jiraAuthID = $jira.login($credential.UserName.TrimStart("\"),[System.Runtime.InteropServices.marshal]::PtrToStringAuto($BSTR))
 [System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($BSTR);
`

