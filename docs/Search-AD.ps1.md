---
Author: p sczepanski
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3788
Published Date: 2013-11-26t14
Archived Date: 2016-03-25t09
---

# search-ad - 

## Description

search-ad without need to install quest active role commandlets or install active directory management gateway.

## Comments



## Usage



## TODO



## function

`test-objectpath`

## Code

`#
 #
 
 Add-Type @'
 using System;
 
 namespace redtoo {
     namespace AD {
         public enum Provider : int {
             LDAP,
             GC,
         }
     }
     public class ADsPathPart {
             public string Provider;
             public string Server;
             public string BaseDN;
             public string RDN;
             public string DCComponent;
             public string Parent;
             public string DistinguishedName;
     }
 }
 '@
 
 function Test-ObjectPath {
 <#
     .SYNOPSIS
         Tests if an Object in AD exists.
 
     .DESCRIPTION
         Tests if an Object in AD exists. Valid inputs are a DN or a DirectoryEntry object.
         With parameter you can force an imput type. Without a parameter the script figures out if an input was a string or directoryEntry object.
 
     .PARAMETER  DistinguishedName
         The distinguished name of the object to be tested. The DN may be specified using aDSPath or distinguished name.
         If an incorrect DN syntax is supplied, it will return $false
 
     .PARAMETER  DirectoryEntry
         A DirectoryEntry object to be tested. 
         If a string value is passed, it will be casted to a DirectoryEntry.
 
     .PARAMETER  isValid
         Verifies that the syntax of the DN is correct. 
         Returns $true if valid and otherwise $false
 
     .PARAMETER  PassThru
         Returns the [DirectoryServices.DirectoryEntry] object if object exists, otherwise returns false
     
     .PARAMETER  AsDN
         Use together with -PassThru. Returns distinguished path as String
 
     .PARAMETER  AsADsPath
         Use together with -PassThru. Returns ADsPath as String
 
     .PARAMETER  $SplitParts
         Returns a PowerShell object with the different LDAP URI parts
 
     .EXAMPLE
         PS C:\> Test-ObjectPath  "CN=myOU,DC=test,DC=intra"
         
         -------------------------------------------------------
         
         Test if an object with the DN CN=myOU,DC=test,DC=intra exists.
 
     .EXAMPLE
         PS C:\> Test-ObjectPath  "CN=myOU,DC=test,DC=intra" -isValid
         
         -------------------------------------------------------
         
         Test if the DN CN=myOU,DC=test,DC=intra is correct.
 
     .EXAMPLE
         PS C:\> Test-ObjectPath -DirectoryEntry ([DirectoryServices.DirectoryEntry]"LDAP://CN=myOU,DC=test,DC=intra"
     
         -------------------------------------------------------
         
         Test if an object represented by the directoryEntry of CN=myOU,DC=test,DC=intra exists
 
     .EXAMPLE
         PS C:\> Test-ObjectPath "LDAP://CN=myOU,DC=test,DC=intra"
         
         -------------------------------------------------------
         
         Test if an object with the DN CN=myOU,DC=test,DC=intra exists.
 
     .EXAMPLE
         PS C:\> Test-ObjectPath ([DirectoryServices.DirectoryEntry]"LDAP://CN=myOU,DC=test,DC=intra"
     
         -------------------------------------------------------
         
         Test if an object represented by the directoryEntry of CN=myOU,DC=test,DC=intra exists
 
     .INPUTS
         System.String,DirectoryServices.DirectoryEntry
 
     .OUTPUTS
         System.Bool
 
     .Notes
         NAME:      Test-ObjectPath
         AUTHOR:    Patrick Sczepanski (patrick@sczepanski.com)
         Version:   20120716
 
 #>
     
     [Cmdletbinding(DefaultParameterSetName="String")]
 
     Param(
         [Alias('dn')]
         [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Mandatory=$True,Position=0,ParameterSetName="String")]
         [string]$DistinguishedName,
         
         [Parameter(ParameterSetName="String")]
         [switch]$isValid,
         
         [Parameter(ValueFromPipelineByPropertyName=$true,Mandatory=$True,Position=0,ParameterSetName="DirectoryEntry")]
         [System.DirectoryServices.DirectoryEntry]
         $DirectoryEntry,
 
         [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Mandatory=$True,Position=0,ParameterSetName="SearchResult")]
         [AllowNull()]
         [DirectoryServices.SearchResult]
         $SearchResult,
         
         [Parameter(ParameterSetName="String")]
         [string]
         $connection,
         
         [Parameter()]
         [switch]
         $Passthru,
 
         [Parameter()]
         [switch]
         $AsDN,
 
         [Parameter()]
         [switch]
         $AsADsPath,
         
         [Parameter()]
         [switch]
         $SplitParts
     )
     Begin {
         $ThisFunctionName = $MyInvocation.MyCommand.Name
         if( ( $PSBoundParameters.ContainsKey("AsDN") -or $PSBoundParameters.ContainsKey("AsADsPath") -or $PSBoundParameters.ContainsKey("SplitParts") ) -and (-not $PSBoundParameters.ContainsKey("Passthru") ) ) {
             Write-Warning "[$ThisFunctionName] :: You need to specify -Passthru if you want to use -AsDN and -AsADsPath"
         }
         function Split-ADSPath ( $ADSPath ) {
             if ( $ADSPath -match '^(?:(?:(?<Provider>LDAP|GC)://)(?:(?<DC>(?:[\w\.-])+)/)*)+(?<Base>(?:(?:URI|CN|OU)=.*?,)*)(?<DCComponent>(?:DC=.*)+)$' ) {
                 New-Object redtoo.ADsPathPart -Property @{
                     "Provider"    = if ($Matches.Contains("Provider") )    { $Matches.Provider };
                     "Server"      = if ($Matches.Contains("DC") )          { $Matches.DC };
                     "BaseDN"      = if ($Matches.Contains("Base") )        { $Matches.base -replace ",$","" };
                     "RDN"         = if ($Matches.Contains("Base") )        { ($Matches.base.split(","))[0] };
                     "DCComponent" = if ($Matches.Contains("DCComponent") ) { $Matches.DCComponent };
                     "Parent"      = if ($Matches.Contains("Provider") )    { ($Matches.base.split(",", [StringSplitOptions]::RemoveEmptyEntries) | select -Skip 1) -join "," };
                     "DistinguishedName" = if ($Matches.Contains("Base") -and `
                                         $Matches.Contains("DCComponent") ) { "{0}{1}" -f $Matches.base, $Matches.DCComponent  };
                 }
             } else {
                 Write-Warning "[$ThisFunctionName] :: Failed to match LDAP Path: '$LDAPPath'"
             }
         }
     }
     Process {
         switch ($pscmdlet.ParameterSetName) {
             "String"    {
                 if ( $connection ) {
                       $connection = $connection + "/"
                 } else {
                     $connection = ""
                 }
                 switch  -regex ( $DistinguishedName ) { 
                     #'^((?<Provider>(LDAP|GC)://)(?<DC>([\w\.-])+/)*)+(?<Base>((URI|CN|OU)=.*?,)*)(?<DCComponent>(DC=.*)+)$'
                     '^(((LDAP|GC)://)(([\w\.-])+/)*)+((UID|CN|OU)=.*?,)*(DC=.*)+$' {
                         $LDAPpath = $DistinguishedName
                         break
                     }
                     '^(([\w\.-])+/)*((UID|CN|OU)=.*?,)*(DC=.*)+$' {
                         $LDAPpath = "LDAP://$connection$DistinguishedName"
                         break 
                     }
                     '^WinNT://(?<Computer>\w.*|\.)/(?<Account>\w.*)$' {
                         $LDAPpath = $DistinguishedName
                         break
                     }
                     Default { return $false }
                 }
                 if ( $isValid ) { 
                     return $true 
                 }
                 $exists = try { [DirectoryServices.DirectoryEntry]::exists("$LDAPpath") } catch { $false }
                 if ( $exists ) {
                     if ( $Passthru ) {
                         if ( $AsDN ) {
                             return ([regex]::Match($LDAPpath,".*/(.*)")).Groups[1].Value
                         } elseif ( $AsADsPath ) {
                             return $LDAPpath 
                         } elseif ( $SplitParts ) {
                             Split-ADSPath $LDAPpath 
                         } else {
                             return  [DirectoryServices.DirectoryEntry]"$LDAPpath"
                         }
                     } else {
                         return $true
                     }
                 } else {
                     return $false
                 }
                 break 
             }
             "DirectoryEntry" {
                 if ( [string]::IsNullOrEmpty( ( $DirectoryEntry | Get-Member -MemberType Property ) ) ) {
                     return $false
                 } else {
                     if ( $Passthru ) {
                         if ( $AsDN ) {
                             return ([regex]::Match($DirectoryEntry.adspath,".*/(.*)")).Groups[1].Value 
                         } elseif ( $AsADsPath ) {
                             return $DirectoryEntry.adspath 
                         } elseif ( $SplitParts ) {
                             Split-ADSPath $DirectoryEntry.adspath
                         } else {
                             return $DirectoryEntry
                         }
                     } else {
                         return $true
                     }
                 }
                 break
             }
             "SearchResult" {
                 if ( [string]::IsNullOrEmpty( $SearchResult ) ) {
                     return $false
                 } else {
                     $DN = $SearchResult.Properties.adspath
                     if ( $Passthru ) {
                         if ( $AsDN ) {
                             return ([regex]::Match($SearchResult.Properties.adspath[0],".*/(.*)")).Groups[1].Value 
                         } elseif ( $AsADsPath ) {
                             return $SearchResult.Properties.adspath[0] 
                         } elseif ( $SplitParts ) {
                             Split-ADSPath ( $SearchResult.Properties.adspath[0] )
                         } else {
                             return  [DirectoryServices.DirectoryEntry]$SearchResult.Properties.adspath
                         }
                     } else {
                         return $true
                     }
                 break
                 }
             }
         }
     }
     End {
     
     }
 }
 
 function Search-AD  {
     <#
         .Synopsis 
             Searching Active Directory
             
         .Description
             Searching Active Directory
 
         .Parameter Provider
             Provider to use to connect. Allowed are GC and LDAP
             Default: LDAP
             
         .Parameter Connection
             Optional domain controller name used to connect to execute the search
             Default: Any (closest)
         
         .Parameter Forest
             Use forest root DN as default searchbase, GC as default provider 
             and set the default scope to subtree
         
         .Parameter Domain
             Use current domain root DN as default searchbase 
             and set the default scope to subtree
         
         .Parameter Searchbase
             Distinguished name of the searchbase to start the search from
             Default: Current Domain
         
         .Parameter Filter
             LDAP filter
             Default: (objectclass=*)
         
         .Parameter Attributes
             Attributes to be returned
             Default distinguishedname
         
         .Parameter Scope
             Search scope. Allowed scopes are base, onelevel, subtree
             Default: base
         
         .Parameter PageSize
             Number of objects returned per page. In standard AD user a number below 1000 which is the default maximum object returned in one step
             Default: 1000
         
         .Parameter SizeLimit
             Maximum number of objects returned. Enter 0 for unlimited numer
             Default: 1000
         
         .Parameter ChooseItem 
             Allows to choose a single item out of multiple items returned by the search
         
         .Parameter PropertyNamesOnly 
             Returns only the property names without values
         
         .Parameter FindOne 
             Returns the first object matching
         
         .Example
             $Group = Search-AD -connection "DC1.mydomain.com" -filter "(&(objectcategory=group)(samaccountname=group51)))" -searchbase "DC=mydomain,DC=com" -scope "subtree"
         
         .OUTPUTS
             [System.DirectoryServices.SearchResultCollection]
             
         .INPUTS
             NA
             
         .Link
             Search-AD
         
         .Notes
             NAME:      Search-AD
             AUTHOR:    Patrick Sczepanski (patrick@sczepanski.com)
             Version:   20120709
     #>
     [Cmdletbinding(DefaultParameterSetName="DN")]
     Param (
         [Parameter(ValueFromPipelineByPropertyName=$true,Position=0)]
         [redtoo.AD.Provider]$Provider   = [redtoo.AD.Provider]::LDAP, 
 
         [Parameter(ValueFromPipelineByPropertyName=$true)]
         [string]
         $Connection,
 
         [Alias('base','b')]
         [Parameter(ValueFromPipelineByPropertyName=$true,Position=0,ParameterSetName="DN")]
         [ValidateScript( {Test-ObjectPath $_ -IsValid} )]
         [string]
         $Searchbase = ([DirectoryServices.DirectoryEntry]"LDAP://RootDSE").DefaultNamingContext,
 
         [Parameter(ValueFromPipelineByPropertyName=$true,ParameterSetName="Domain")]
         [switch]
         $DomainBase,
         
         [Parameter(ValueFromPipelineByPropertyName=$true,ParameterSetName="Forest")]
         [switch]
         $ForestBase,
         
         [Alias('f')]
         [Parameter(ValueFromPipelineByPropertyName=$true)]
         [string]
         $Filter        = "(objectclass=*)",
 
         [Alias('a','attribute','attrib')]
         [Parameter(ValueFromPipelineByPropertyName=$true)]
         [string[]]
         $Attributes,
         
         [Alias('s')]
         [Parameter(ValueFromPipelineByPropertyName=$true)]
         [DirectoryServices.SearchScope]
         $Scope         = "base", 
 
         [Parameter(ValueFromPipelineByPropertyName=$true)]
         [int32]$PageSize = 1000,
         
         [Parameter(ValueFromPipelineByPropertyName=$true)]
         [int32]$SizeLimit = 1000,
         
         [Parameter(ValueFromPipelineByPropertyName=$true)]
         [switch]$PropertyNamesOnly,
 
         [Alias('choose','select','c')]
         [Parameter(ValueFromPipelineByPropertyName=$true)]
         [switch]
         $ChooseItem,
         
         [Alias('One')]
         [switch]
         $FindOne,
         
         [switch]
         $CountOnly,
         
         [Alias('Chasing')]
         [Parameter(ValueFromPipelineByPropertyName=$true)]
         [DirectoryServices.ReferralChasingOption]
         $ReferralChasing = [DirectoryServices.ReferralChasingOption]::None,
         
         [DirectoryServices.SecurityMasks]
         $SecurityMasks = [DirectoryServices.SecurityMasks]::Dacl
         
     ) 
     Begin {
         $ThisFunctionName = $MyInvocation.MyCommand.Name
         $RootDSE = [DirectoryServices.DirectoryEntry]"LDAP://RootDSE"
         if ( ! $RootDSE ) {
             Write-Warning "[$ThisFunctionName] :: You are not connected to AD. This function can only be used within an AD Forest."
             Return $null
         }
     }
     Process {
         switch ( $psCmdlet.ParameterSetName ) {
             "Forest" {  
                 $SearchBase = $RootDSE.rootDomainNamingContext
                 if ( ! $PSBoundParameters.ContainsKey("Provider") ) {
                     $Provider = [redtoo.AD.Provider]::GC; 
                 } 
                 if ( ! $PSBoundParameters.ContainsKey("Scope") ) {
                     $Scope = [DirectoryServices.SearchScope]::Subtree
                 }
              }
             "Domain" { 
                 $SearchBase = $RootDSE.defaultNamingContext
                 if ( ! $PSBoundParameters.ContainsKey("Scope") ) {
                     $Scope = [DirectoryServices.SearchScope]::Subtree
                 }            
             }
         }
         $SearchBaseParts = Test-ObjectPath $SearchBase -passthru -SplitParts
         if ( $Connection ) {
             $SearchBaseParts.Server = $Connection
         } else {
             
             $DomainDNS = ( $SearchBaseParts.DCComponent ) -replace "DC=","" -replace ",","."
             $SearchBaseParts.Server =  ( [adsi]"LDAP://$DomainDNS/RootDSE" ).dnsHostName
         }
         
         if ( $Provider ) {
             $SearchBaseParts.Provider = $Provider
         }
         $SearchBase = "{0}://{1}/{2}" -f $SearchBaseParts.Provider, $SearchBaseParts.Server, $SearchBaseParts.DistinguishedName
         
         if ( $attributes -notcontains "distinguishedname" ) {
             $attributes +=  "distinguishedname"
         }
         
         if ( $CountOnly ) {
             $PropertyNamesOnly = $true
             $cacheResults = $false
         } else {
             $cacheResults = $true
         }
         [DirectoryServices.DirectoryEntry]$searchbaseURI = $SearchBase
         [DirectoryServices.DirectorySearcher]$Searcher = new-object DirectoryServices.DirectorySearcher($searchbaseURI)
         if ( $attributes -match "NTSecurityDescriptor" ) {
             $Searcher.SecurityMasks = $SecurityMasks
         }
         $Searcher.filter = $filter
         $Searcher.CacheResults = $cacheResults
         $Searcher.SearchScope = $scope
         $Searcher.PageSize = $PageSize
         $Searcher.PropertyNamesOnly = $PropertyNamesOnly
         $Searcher.PropertiesToLoad.AddRange($attributes)
         $Searcher.ReferralChasing = $ReferralChasing
         try {
             if ( $FindOne ) {
                 [System.DirectoryServices.SearchResult]$result = $searcher.FindOne()
             } else {
                 [System.DirectoryServices.SearchResultCollection]$result = $searcher.FindAll()
             }
         } 
         catch {
             Write-Warning "[$ThisFunctionName] :: $_`nBase:       $SearchBase`nFilter:      $filter`nSearchScope: $scope`nAttributes:  $Attributes " 
             return $null
         }
         switch ( @(,$result) ) {
             { ($_ -is [System.DirectoryServices.SearchResultCollection]) -and ($_.count -lt 1) } { Write-Debug "[Search-AD] :: Cannot find an object in $searchbase using filter $filter"
                 if (  $CountOnly ) {
                     return 0
                 } else {
                     return $null
                 }
                 break }
             { ($_ -is [System.DirectoryServices.SearchResult] -or ($_.count -eq 1)) } { Write-Debug "[Search-AD] :: Found 1 Object"
                 if (  $CountOnly ) {
                     return 1
                 } else {
                     return $result
                 }
                 break }
             { ($_ -is [System.DirectoryServices.SearchResultCollection]) -and ($_.count -gt 1) } {
                 if (  $CountOnly ) {
                     return $_.count
                 } else {
                     if ( $chooseitem ) {
                         [int]$count = -1
                         foreach($object in $result) {
                             $count = $count + 1
                             Write-Host "[$count]: " $object.Properties.distinguishedname
                         }
                         $selection = Read-Host "Please select object"
                         if ( $($selection -lt 0) -or $($selection -gt $count) -or $($count -isnot [int])  ) { Write-Error "Selection '$selection' out of scope.`r`nPlease enter an integer value between 0 and $count."; exit(0)  }
                         return $result[$selection]
                     } else {
                         return $result
                     }
                 }
                 break }
             default { Write-Error -message "[$ThisFunctionName] :: Issue with switch statement. Please check code. Unexpected Error."; }
         }
     }
     End {
 
     }
 }
`

