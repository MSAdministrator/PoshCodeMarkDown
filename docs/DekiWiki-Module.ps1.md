---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 1.5
Encoding: ascii
License: cc0
PoshCode ID: 692
Published Date: 
Archived Date: 2010-05-08t05
---

# dekiwiki module - 

## Description

the first of many script cmdlets for working with dekiwiki

## Comments



## Usage



## TODO



## module

`set-dekiurl`

## Code

`#
 #
 ####################################################################################################
 ##
 ####################################################################################################
 ####################################################################################################
 $hr = Add-Module HttpRest -Passthru
 Add-Module "$PSScriptRoot\Utilities.ps1"
 
 $url = "http://powershell.wik.is"
 $api = "$url/@api/deki"
 
 
 FUNCTION Set-DekiUrl {
    PARAM ([uri]$baseUri=$(Read-Host "Please enter the base Uri for your RESTful web-service"))
    Set-HttpDefaultUrl $baseUri
 }
 
 FUNCTION Set-DekiCredential {
    PARAM($Credential=$(Get-Credential -Title "Http Authentication Request - $($global:url.Host)" `
                                       -Message "Your login for $($global:url.Host)" `
                                       -Domain $global:url.Host ))
    Set-HttpCredential $Credential
 }
 
 New-Alias e2 Encode-Twice -EA "SilentlyContinue"
 
 
 Add-Type '
 public class ModuleInfo {
    public string Name;
    public string[] Author;
    public string CompanyName;
    public string[] Copyright;
    public string[] Description;
    public System.Version ModuleVersion;
    public string[] RequiredAssemblies;
    public string[] Dependencies;
    public System.Guid GUID = System.Guid.NewGuid();
    public string[] PowerShellVersion;
    public string[] ModulesToProcess;
    public System.Version CLRVersion;
    public string[] FormatsToProcess;
    public string[] TypesToProcess;
    public string[] OtherItems;
    public string ModuleFile;
 }
 ' -EA "SilentlyContinue"
 
 CMDLET Get-DekiSiteMap {
 PARAM( 
 	[Parameter(Position=0, Mandatory=$false)]
 	[string]
 	$StartPage
 ,
 	[Parameter(Position=1, Mandatory=$false)]
 	[ValidateSet("xml","html","google")]
 	[string]
 	$Format = "xml"
 )
    if($StartPage) {
       Invoke-Http GET "pages/=$(e2 $StartPage)/tree" @{"format"=$format} | Receive-Http -Out Xml
    } else {
       Invoke-Http GET "pages" @{"format"=$format} | Receive-Http -Out Xml
    }
 }
 
 
 CMDLET Get-DekiContent {
 PARAM(
    [Parameter(Position=0, Mandatory=$true)]
    [string]
    $pageName
 , 
    [Parameter(Position=1, Mandatory=$false)]
    [int]
    $section
 , 
    [Parameter(Position=5, Mandatory=$false)]
    [ValidateSet("edit", "raw", "view", "viewnoexecute")]
    [string]
    $mode="view"
 )
    Invoke-Http "GET" "pages/=$(e2 $pageName)/contents" @{mode=$mode;section=$section} | Receive-Http -Out Xml
 }
 
 CMDLET Get-DekiFile {
 PARAM(
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [string]
    $pageName
 ,
    [Parameter(Position=1, Mandatory=$false, ValueFromPipelineByPropertyName=$true)]
    [string]
    $fileName
 ,
    [Parameter(Position=2, Mandatory=$false)]
    [string]
    $destination
 )
 PROCESS {
    $files = Invoke-Http "GET" "pages/=$(e2 $pageName)/files" | Receive-Http Xml //file
    
    Write-Verbose "Fetching $($fileName) from the $($files.Count) in $pageName"
    if(!$fileName) {
       write-output $files | Add-Member NoteProperty PageName $pageName -passthru
    } else {
       foreach($file in $($files | where { $_.filename -like $fileName } )) {
          Invoke-Http "GET" "pages/=$(e2 $pageName)/files/=$(e2 $file.filename)" | Receive-Http File $(Get-FileName $file.filename $destination)
       }
    }
 }
 }
 
 CMDLET Set-DekiContent {
 PARAM(
    [Parameter(Position=0, Mandatory=$true)]
    [string]
    $pageName
 , 
    [Parameter(Position=1, Mandatory=$true, 
                           ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [string]
    $content
 , 
    [Parameter(Position=2, Mandatory=$false)]
    $section
 ,
    [switch]$new
 ,
    [switch]$append
 )
    $with = @{"edittime"=$(Get-UtcTime "yyyyMMddhhmmss")}
    
    if($new) {
       $with.abort = "exists"
    }
    if($append) {
       $with.section = "0"
    } elseif($section) {
       $with.section = $section
    }
 
    $result = Invoke-Http POST "pages/=$(e2 $pageName)/contents" -With $with -Content $content -Auth $true -Wait
    if($result.IsSuccessful) {
       return "$url/$($result | Receive-Http Text //edit/page/path)"
    } else {
       return $result
    }
 }
 
 CMDLET New-DekiContent {
 PARAM(
    [Parameter(Position=0, Mandatory=$true)]
    [string]$pageName
 , 
    [Parameter(Position=1, Mandatory=$true, 
               ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [string]
    $content
 )
 PROCESS {
    Set-DekiContent $pageName $content -new
 }
 }
 
 CMDLET Set-DekiFile {
 param(
    [Parameter(Position=0, Mandatory=$true)]
    [string]$pageName
 , 
    [Parameter(Position=1, Mandatory=$true,
               ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    $Files
 )
    $hasDescriptions = $false;
    if($files -is [Hashtable]) {
       $hasDescriptions = $true;
       $fileNames = $files.Keys
    } else {
       $fileNames = $files
    }
    
    foreach($fileName in $fileNames) {
       foreach($file in (gci $fileName)) {
          $with = @{}
          if($hasDescriptions) {
             $With.description = $files[$fileName]
          }
 
          $result = Http-Invoke PUT "pages/=$(e2 $pageName)/files/=$($file.Name)" $With $file.FullName -Wait
          if($result.IsSuccessful) {
             return ($result | Receive-Http Text //file/href)
          } else {
             return $result.Response
          }
       }
    }
 }
 
 
 CMDLET Move-DekiContent {
 PARAM(
    [Parameter(Position=0, Mandatory=$true)]
    [string]
    $pageName
 , 
    [Parameter(Position=1, Mandatory=$true)]
    [string]
    $newPageName
 )
    $with = @{"to"="$newPageName"}
    
    $result = Invoke-Http POST "pages/=$(e2 $pageName)/move" -With $with -Auth $true -Wait
    if($result.IsSuccessful) {
       return "$url/$($result | Receive-Http Text '//page[1]/path' )"
    } else {
       return $result
    }
 }
 
 CMDLET New-DekiAlias {
 PARAM(
    [Parameter(Position=0, Mandatory=$true)]
    [string]
    $pageName
 , 
    [Parameter(Position=1, Mandatory=$true)]
    [string]
    $Alias
 )
   
    $result = Invoke-Http POST "pages/=$(e2 $pageName)/move" @{"to"="$Alias"} -Auth $true -Wait
    if($result.IsSuccessful) {
       $aliasPage = "$url/$($result | Receive-Http Text '//page[1]/path' )"
       $result = Invoke-Http POST "pages/=$(e2 $Alias)/move" @{"to"="$pageName"} -Auth $true -Wait
       if($result.IsSuccessful) {
          return $aliasPage
       } else {
          return $result
       }
    } else {
       return $result
    }
 }
 
 CMDLET Remove-DekiContent {
 PARAM(
    [Parameter(Position=0, Mandatory=$true,ValueFromPipelineByPropertyName=$true)][string]$pageName
 ,
    [Parameter(Position=2, Mandatory=$false)][switch]$recurse
 )
 
    Http-Invoke DELETE "pages/=$(e2 $pageName)" @{recursive=$($recurse.ToBool())}
    if($result.IsSuccessful) {
       return "DELETED $url/$PageName"
    } else {
       return $result.Response
    }
 }
 
 CMDLET Remove-DekiFile {
 PARAM(
    [Parameter(Position=0, Mandatory=$true,ValueFromPipelineByPropertyName=$true)][string]$pageName
 ,
    [Parameter(Position=1, Mandatory=$true,
               ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("Name")]
    $fileNames
 )
    $files = Invoke-Http GET "pages/=$(e2 $pageName)/files" | Receive-Http Text //file/filename
    foreach($fileName in $fileNames) {
       foreach($file in $($files -like $fileName)) {
          $path = "pages/=$(e2 $pageName)/files/=$(e2 $file)"
          Write-Host $path -fore cyan
          $result = Invoke-Http DELETE $path -Auth $true -Wait
       
          if($result.IsSuccessful) {
             "DELETED: $api/$path"
          } else {
             $result.Response
          }
       }
    }
 }
 
 Export-ModuleMember Set-DekiCredential, Set-DekiUrl, 
                     Get-HtmlHelp, Get-DekiFile, Get-DekiSiteMap, Get-DekiContent, 
                     Set-DekiContent, New-DekiContent,
                     Move-DekiContent, New-DekiAlias,
                     Remove-DekiContent, Remove-DekiFile,
                     Set-DekiFile
`

