---
Author: axel limousin
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: utf-8
License: cc0
PoshCode ID: 4497
Published Date: 2016-09-29t09
Archived Date: 2016-09-07t03
---

# draw nested groups - 

## Description

checking security group nesting strategy (ie

## Comments



## Usage



## TODO



## script

`dig-adsecuritygroupmembers`

## Code

`#
 #
 <#
 #######
 Syntax
 #######
 
 List-ADSecurityGroupMembers <ObjectDN> [-SC {<Onelevel> | <Subtree>}] [-RL <integer>] | 
 Graph-ADSecurityGroupMembers > <FilePath>
 
 #########
 Examples
 #########
 
 Graph-ADSecurityGroupMembers > "$env:USERPROFILE\Desktop\ADSecurityGroupMembers.viz"
 
  same as
 
 List-ADSecurityGroupMembers "OU=MyOU,DC=MyDom,DC=MyRoot"  -SC Subtree -RL 5 | 
 Graph-ADSecurityGroupMembers > "$env:USERPROFILE\Desktop\ADSecurityGroupMembers.viz"
 
 With default Scope (-SC) "Subtree" and default RecursionLevel (-RL) of 5, it draws Group Nesting for all Security Groups in "MyOU" and children OUs digging through 5 nesting levels.
 
 Graph-ADSecurityGroupMembers > "$env:USERPROFILE\Desktop\ADSecurityGroupMembers.viz"
 
  It draws Group Nesting for the Security Groups "MyGroup" digging through 5 nesting levels (default RecursionLevel).
 
 ######
 Usage
 ######
 
 ADSecurityGroupMembers generates ADSecurityGroupMembers.viz file. Install Graphviz (www.graphviz.org) and associate "gvedit.exe" to .viz extension. With Graphiz GUI App, in "Graph>Settings" menu, you have the choice between "dot", "fdp", "sfdp", "circo", "neato", "osage" or "twopi" models to gen a .pdf, .svg or .png graphic. You should try "dot" model first (default). Domain Local Group graph cell border is colored in red, Global in green, Universal in cyan.
 
 #################
 Steps suggestion
 #################
 
 - Install Graphviz from www.graphviz.org
 - Add the program path �C:\Program Files (x86)\Graphviz2.30\bin;� to Path environment variable
 - In PowerShell console, verify that dot.exe is available just by executing dot �help
 - In PowerShell console, paste script code and complement with code below:
 
  $Output = List-ADSecurityGroupMembers $ADObj | Graph-ADSecurityGroupMembers 
  $Output | Out-File -FilePath "$env:USERPROFILE\Desktop\NestedGroups.viz" -Encoding ASCII
  dot -Tjpg "$env:USERPROFILE\Desktop\NestedGroups.viz" -o "$env:USERPROFILE\Desktop\NestedGroups.jpg"
  del "$env:USERPROFILE\Desktop\NestedGroups.viz"
  Where $ADObj is a distinguishedName of a domain, an ou, a container or a group. The process can be long if you handle hundred of nested groups.
 
 #######
 Script
 #######
  
 =====================================================================================================
 PURPOSE:	Graph Nested AD Security Groups by Member Property
 http://gallery.technet.microsoft.com/scriptcenter/Graph-Nested-AD-Security-20aa3fcd
 
 AUTHORS:	Axel Limousin
 VERSION:	1.0 
 DATE:	09/28/2013 
 
 THANKS:	Thomas Corbiere
 =====================================================================================================
 #>
 
 $ErrorActionPreference="SilentlyContinue"
 
 function Dig-ADSecurityGroupMembers
 {
 	param
 	(
 		[Parameter(ValueFromPipeline = $true, Mandatory = $true)]
 		[ValidateScript({$_.GroupCategory -eq "Security"})]
 		[Microsoft.ActiveDirectory.Management.ADGroup] $ADSGroup,
 		
 		[Parameter(ValueFromPipeline = $false, Mandatory = $false)]
 		[Alias("RL")]
 		[Byte] $RecursionLevel = 5,
 		
 		[Parameter(ValueFromPipeline = $false, Mandatory = $false)]
 		[Alias("CL")]
 		[Byte] $CurrentLevel = 0
     	)
  
     	if ($CurrentLevel -ge $RecursionLevel)
 	{  
 		return
     	}
  
 	$ADSGroupMbrs = Get-ADGroupMember $ADSGroup.DistinguishedName | 
 	Where-Object -FilterScript { (Get-ADGroup $_).GroupCategory -eq "Security" }
 
 	$ADSGObj = New-Object PSObject -Property @{
 		Name			= $ADSGroup.Name
 		DistinguishedName	= $ADSGroup.DistinguishedName
     		GroupScope		= $ADSGroup.GroupScope
     		Members			= $ADSGroupMbrs | Select-Object distinguishedName
 		}
 	
 	$ADSGObj
 	
     	$ADSGroupMbrs | ForEach-Object -Process {
 	
 		Get-ADGroup $_.DistinguishedName | Dig-ADSecurityGroupMembers -RL $RecursionLevel -CL ($CurrentLevel+1)
 	}
 }
 
 function List-ADSecurityGroupMembers
 {
 	param
 	(
 		[Parameter(ValueFromPipeline = $false, Mandatory = $false)]
 		[String] $ADDN = (Get-ADDomain).DistinguishedName,
 		
 		[Parameter(ValueFromPipeline = $false, Mandatory = $false)]
 		[Alias("SC")]
 		[Microsoft.ActiveDirectory.Management.ADSearchScope] $Scope = "Subtree",
 		
 		[Parameter(ValueFromPipeline = $false, Mandatory = $false)]
 		[Alias("RL")]
 		[Byte] $RecursionLevel = 5
     	)
  	
 	$ADObject = Get-ADObject $ADDN -Properties groupType
 	
 	if ($ADObject.ObjectClass -eq "domainDNS" -or $ADObject.ObjectClass -eq "organizationalUnit" -or $ADObject.ObjectClass -eq "container")
 	{ 
 		$ADSGroupList = Get-ADGroup -Filter {GroupCategory -eq "Security"} -SearchBase $ADDN -SearchScope $Scope | 
 		ForEach-Object -Process {
 		
 			$_ | Dig-ADSecurityGroupMembers -RL $RecursionLevel 
     		} 
     
 		$ADSGroupList = $ADSGroupList | Sort-Object -Unique -Property DistinguishedName
     
 		return $ADSGroupList 
 	}
 	
 	elseif ($ADObject.ObjectClass -eq "group" -and $ADObject.groupType -like "-2*")
 	{
 		$ADSGroup = Get-ADGroup $ADObject.DistinguishedName
 		
 		$ADSGroupList = Dig-ADSecurityGroupMembers $ADSGroup -RL $RecursionLevel
 		
 		$ADSGroupList = $ADSGroupList | Sort-Object -Unique -Property DistinguishedName 
 		
 		return $ADSGroupList
 	}
 	
 	else
 	{
 		Write-Warning -Message "Please select a Domain, an OU, a Container or a Security Group" 
 	}
 }
 
 function Graph-ADSecurityGroupMembers
 {
 	param
 	(
 		[Parameter(ValueFromPipeline = $true, Mandatory = $true)]
 		[PSObject[]] $ADSGroupList,
 		
 		[Parameter(ValueFromPipeline = $false, Mandatory = $false)]
 		[String] $DomainLocalColor = "Red",
 		[Parameter(ValueFromPipeline = $false, Mandatory = $false)]
 		[String] $GlobalColor      = "Green",
 		[Parameter(ValueFromPipeline = $false, Mandatory = $false)]
 		[String] $UniversalColor   = "Cyan"
     	)
 
 	begin
 	{
 		$Groups = @()
 
 		$GroupColor = {
             
 			param
 			(	
 				[Parameter(ValueFromPipeline = $false, Mandatory = $true)]
 				[Microsoft.ActiveDirectory.Management.ADGroupScope] $GroupScope
 			)
 
             		switch ($GroupScope)
 			{
 				"DomainLocal" { return $DomainLocalColor }
 				"Global"      { return $GlobalColor }
 				"Universal"   { return $UniversalColor }
             		}
         		}
 
         		$GraphNode = {
 		
 			param
 			(
 				[Parameter(ValueFromPipeline = $false, Mandatory = $true)]
 				[PSObject] $ADSGroup
 			)
 		
 			$StrMark = $ADSGroup.DistinguishedName.IndexOf("DC=")
 			$DomainName = $ADSGroup.DistinguishedName.Substring($StrMark)
 
             		"node_{0} [" -f [Array]::indexOf($Groups,$ADSGroup.DistinguishedName) | Write-Output
             		'label="{0}|{1}";' -f $ADSGroup.Name, $DomainName | Write-Output
             		"color={0};" -f (&$GroupColor $ADSGroup.GroupScope) | Write-Output
             		'shape = "record"' | Write-Output
             		"]" | Write-Output
         		}
 
         		"digraph g {" | Write-Output
         		'graph [rankdir = "LR"] ;' | Write-Output
         		"overlap = false ;" | Write-Output
     	}
 
  	process
 	{
 		foreach ($ADSGroup in $ADSGroupList)
 		{
 			if ([Array]::indexOf($Groups,$ADSGroup.DistinguishedName) -eq -1)
 			{
 				$Groups += $ADSGroup.DistinguishedName
             			&$GraphNode $ADSGroup
 			}
 
 			$ADSGroup.Members | 
 			ForEach-Object -Process {
 				
 				if ([array]::indexOf($Groups,$_.distinguishedName) -eq -1)
 				{
 					$Groups += $_.distinguishedName
 					$ChildGroup = Get-ADGroup $_.distinguishedName
             				&$GraphNode $ChildGroup
        				}
 
             			'node_{0} -> node_{1} [dir= "back"] ;' -f [Array]::indexOf($Groups,$ADSGroup.DistinguishedName), [Array]::indexOf($Groups,$_.distinguishedName) | 
 				Write-Output
 			}
 		}
 	}
 
 	end
 	{    
 		"}" | Write-Output
 	}
 }
`

