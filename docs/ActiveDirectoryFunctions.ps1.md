---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3011
Published Date: 2012-10-18t10
Archived Date: 2012-01-14t07
---

# activedirectoryfunctions - 

## Description

a bunch of ad-related functions … i’m only pasting this because i can’t find another get-adcomputer or get-ntaccountname

## Comments



## Usage



## TODO



## function

`get-aduser`

## Code

`#
 #
 #.SYNOPSIS
 function Get-ADUser {
 [CmdletBinding()]
 param([string]$UserName=${Env:userName})
    $ads = New-Object System.DirectoryServices.DirectorySearcher([ADSI]'')
    $ads.filter = "(&(objectClass=Person)(samAccountName=$UserName))"
    $ads.FindAll().GetEnumerator() | %{ $_.GetDirectoryEntry() }
 }
 
 
 function Get-NTAccountName {
 [CmdletBinding()]
 param(
 	[Parameter(ValueFromPipelineByPropertyName=$true)]
 	[string]$Name
 )
 process {
 	$ads = New-Object System.DirectoryServices.DirectorySearcher([ADSI]'')
 	$ads.filter = "(|(name=$Name)(samAccountName=$Name))"
 	$distinguishedName = $ads.FindOne().Properties["distinguishedname"]
 
 	$objTrans = New-Object -comObject "NameTranslate"
 	$objNT = $objTrans.GetType()
 
 	$null = $objNT.InvokeMember("Init", "InvokeMethod", $Null, $objTrans, (3, $Null))
 
 	$null = $objNT.InvokeMember("Set", "InvokeMethod", $Null, $objTrans, (1, "$distinguishedName"))
 
 	$objNT.InvokeMember("Get", "InvokeMethod", $Null, $objTrans, 3)
 }}
 
 
 function Get-SID {
 [CmdletBinding()]
 param(
 	[Parameter(ValueFromPipelineByPropertyName=$true)]
 	[string]$Name
 )
 process {
 	$ads = New-Object System.DirectoryServices.DirectorySearcher([ADSI]'')
 	$ads.filter = "(|(name=$Name)(samAccountName=$Name))"
 	new-object security.principal.securityidentifier $ads.FindOne().Properties["objectSID"][0], 0
 }}
 
 
 #.SYNOPSIS
 function Get-ADComputer {
 [CmdletBinding()]
 param(
 	[Parameter(ValueFromPipelineByPropertyName=$true)]
 	[string]$ComputerName=${Env:ComputerName}
 )
    $ads = New-Object System.DirectoryServices.DirectorySearcher([ADSI]'')
    $ads.filter = "(&(objectClass=Computer)(name=$ComputerName))"
    $ads.FindAll().GetEnumerator() | %{ 
       $Computer = $_.GetDirectoryEntry()
 	  $Computer = Resolve-PropertyValueCollection -InputObject $Computer
       Add-Member -InputObject $Computer -Type NoteProperty -Name SID -Value (new-object security.principal.securityidentifier $Computer.objectSID, 0)
       Add-Member -InputObject $Computer -Type NoteProperty -Name GUID -Value (new-object GUID (,[byte[]]$Computer.objectGUID))
       Add-Member -InputObject $Computer -Type NoteProperty -Name CreatorSID -Value (new-object security.principal.securityidentifier $Computer."mS-DS-CreatorSID", 0)
       Add-Member -InputObject $Computer -Type NoteProperty -Name NTAccountName -Value (Get-NTAccountName $ComputerName)
 	  $Computer
    }
 }
 
 #.SYNOPSIS
 function Get-ADGroup {
 [CmdletBinding()]
 param([string]$UserName)
    $ads = New-Object System.DirectoryServices.DirectorySearcher([ADSI]'')
    $ads.filter = "(&(objectClass=Group)(samAccountName=$UserName))"
    $ads.FindAll().GetEnumerator() | %{ $_.GetDirectoryEntry() }
 }
 
 
 #.SYNOPSIS
 function Get-DistinguishedName { 
 [CmdletBinding()]
 param([string]$UserName)
    (Get-ADUser $UserName).DistinguishedName
 }
 
 #.SYNOPSIS
 #.EXAMPLE
 #.EXAMPLE
 #
 function Get-GroupMembership {
 [CmdletBinding()]
 param([string]$Name,[int]$RecurseLimit=-1)
 
 if(!$Name.StartsWith("CN=","InvariantCultureIgnoreCase")) {
    $Name = Get-DistinguishedName $Name
 }
 
    $groups = ([adsi]"LDAP://$Name").MemberOf
    if ($groups -and $RecurseLimit) {
       Foreach ($gr in $groups) {
          $groups += @(Get-GroupMembership $gr -RecurseLimit:$($RecurseLimit-1) |
                     ? {$groups -notcontains $_})
       }
    }
    return $groups | Convert-DistinguishedName
 }
 
 function Convert-DistinguishedName {
 [CmdletBinding()]
 param(
    [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Position=0)]
    [string]$name
 )
 process {
    if(!$Name.StartsWith("CN=","InvariantCultureIgnoreCase")) {
       $Name = Get-DistinguishedName $Name
    }
    $name -replace "CN=","Name=" -replace "DC=","Domain=" -replace "OU=","Org=" | ConvertFrom-PropertyString -Delimiter "," | ForEach { $_.Domain = $_.Domain -join "."; $_ } | Add-Member NoteProperty DN $name -passthru
 }
 }
 
 function Resolve-PropertyValueCollection { 
 param(
 	[Parameter(ValueFromPipeline=$true)]
 	$InputObject
 )
 process {
 	$SingleMembers = @()
 	$MultiMembers = @()
 	$InputObject | Get-Member -Type Property | ForEach-Object {
 		$Name = $_.Name
 		if($InputObject.($Name).Count -le 1) {
 			$SingleMembers += $Name
 		} else {
 			$MultiMembers += $Name
 		}
 	}
 	
 	$OutputObject = Select-Object -InputObject $InputObject -Property $MultiMembers
 	foreach($member in $singleMembers) {
 		Add-Member -InputObject $OutputObject -Type NoteProperty -Name $Member -Value ($InputObject.$Member)[0]
 	}
 	$OutputObject
 }
 }
 
 
 #. SYNOPSIS
 function Select-UserInfo {
 [CmdletBinding()]
 param(
    [Parameter(Mandatory=$true,Position=0,ParameterSetName="Input",ValueFromPipeline=$true)]
    [System.DirectoryServices.DirectoryEntry[]]$InputObject
 ,
    [Parameter(Mandatory=$true,Position=0,ParameterSetName="Name",ValueFromPipelineByPropertyName=$true)]
    [string[]]$name
 )
 process {
    switch($PSCmdlet.ParameterSetName) {
    "Name" {
       foreach($n in $Name) {
          Write-Verbose "Getting $n User Info"
          Get-ADUser $n | Resolve-PropertyValueCollection
       }
    }
    "Input" {
       foreach($io in $InputObject) {
          Write-Verbose "Converting User Info for $($io.displayName)"
          Resolve-PropertyValueCollection -InputObject $io
       }
    }
    }
 }
 }
 
 
 function Get-GroupMembers {
 [CmdletBinding()]
 param(
 [Parameter(Mandatory=$true,Position=0,ParameterSetName="Input",ValueFromPipeline=$true)]
 [string]$GroupName
 )
 process {
    Foreach ($member in (Get-ADGroup $GroupName).Members() ) {
       new-object System.DirectoryServices.DirectoryEntry $member | Resolve-PropertyValueCollection
    }
 }
 }
`

