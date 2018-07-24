---
Author: david sjstrand
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6386
Published Date: 2016-06-15t16
Archived Date: 2016-06-17t08
---

# content libraries - 

## Description

a small set of powercli functions to work with content libraries. it can’t do subscribed or published content libraries yet, and they lack in feedback (some progress bars would be nice) but i’ll probably add that in the future.

## Comments



## Usage



## TODO



## function

`new-cllocallibrary`

## Code

`#
 #
 function New-CLLocalLibrary
 {
     param(
         [string]$Name, 
         [string]$Datastore, 
         [string]$Description)
     $cisLibrary = Get-CisService com.vmware.content.local_library
      
     [VMware.VimAutomation.Cis.Core.Types.V1.ID]$datastoreID = (Get-Datastore $Datastore).extensiondata.moref.value
     
     
     $createSpec = $CisLibrary.help.create.create_spec.CreateExample()
     $createSpec.server_guid = $null
     $createspec.name = $Name
     $createSpec.description = $Description
     $createSpec.type = "LOCAL"
     $createSpec.publish_info.persist_json_enabled = $false
     $createSpec.publish_info.published = $false
     $createSpec.publish_info.authentication_method = 'NONE'
     $StorageSpec = New-Object PSObject -Property @{
         datastore_id = $datastoreID
         type         = "DATASTORE"
     }
     
     $CreateSpec.storage_backings.Add($StorageSpec)
     $UniqueID = [guid]::NewGuid().Guid
     $CisLibrary.create($UniqueID, $createspec).value
 }
 
 function Get-CLLibrary
 {
 	param([String]$Name = "*", [ValidateSet("LOCAL","SUBSCRIBED","*")][String]$Type = "*")
 	$cis = Get-cisService com.vmware.content.library
 	$cis.list() | ForEach-Object {$cis.get($_)} | Where-Object {$_.Name -like $Name -and $_.Type -like $Type} | Add-Member -TypeName "ContentLibrary" -PassThru
 }
 
 function Get-CLItem
 {
     [cmdletbinding(DefaultParameterSetName='ContentLibrary')]
 	param([Parameter(ValueFromPipeLine=$true,ParameterSetName='ContentLibrary')]$ContentLibrary, 
           [Parameter(ParameterSetName='ContentLibrary')][String]$Name = "*", 
           [Parameter(ParameterSetName='ContentLibrary')][String]$Type = "*", 
           [Parameter(ParameterSetName='Id',Mandatory=$true)][String]$Id)
     BEGIN
     {
 	    $cisItem = Get-cisService com.vmware.content.library.Item
     }
     
     PROCESS
     {
         if($PSCmdlet.ParametersetName -eq 'ContentLibrary')
         {
             Write-Verbose "Parameter set ContentLibrary"
             if ($PSBoundParameters.ContainsKey('ContentLibrary'))
             {
                 Write-Verbose "Single ContentLibrary"
                 if ($ContentLibrary -is [String] -or $ContentLibrary -is [VMware.VimAutomation.Cis.Core.Types.V1.ID])
                 {
                     Write-Verbose "By id"
                     $Libraries = @($ContentLibrary)
                 } else {
                     Write-Verbose "By object"
                     $Libraries = @($ContentLibrary.Id)
                 }
                     
             } else {
                 $Libraries = @( Get-CLLibrary | ForEach-Object { $_.Id } )
             }
             foreach ($cl in $Libraries)
             {
                 Write-Verbose "Getting $cl"
 	            $cisItem.list($cl) | ForEach-Object {$cisItem.get($_)} | Where-Object {$_.Name -like $Name -and $_.Type -like $Type} | Add-Member -TypeName "ContentLibraryItem" -PassThru
             }
         } else {
             $cisItem.get($Id) | Add-Member -TypeName "ContentLibraryItem" -PassThru
         }
     }
 }
 
 
 function Import-CLItem
 {
     [cmdletbinding()]
 	param($ContentLibrary, [String]$Name, [String]$FilePath)
 	$Type = $FilePath -replace '^.*\.',''
 	$cisItem = Get-CisService com.vmware.content.library.item
 	$itemModel = $cisItem.Help.create.create_spec.CreateExample()
 	$itemModel.Name = $Name
 	$itemModel.Library_Id = $ContentLibrary.Id.Value
 	$itemModel.Type = $type
 	$itemId = $cisItem.create([guid]::NewGuid().guid, $itemModel)
 	
     $FileDirectory = Split-Path -Parent (Resolve-path -Path $FilePath)
     $missingFile = @(Split-Path -Leaf $FilePath)
 
     $cisUpdateSession = Get-CisService com.vmware.content.library.item.update_session
 	$updatesessionmodel = $cisUpdateSession.help.create.create_spec.CreateExample()
 	$updatesessionmodel.Library_Item_Id = $itemId
 	$sessionId = $cisUpdateSession.create([guid]::NewGuid().guid,$updatesessionmodel)
 	do
     {
         $uploadfile = $missingFile[-1]
 	    $cisFile = Get-CisService com.vmware.content.library.item.updatesession.file
 	    $fileSpec = $cisFile.Help.add.file_spec.CreateExample()
         $fileSpec.source_endpoint = $null
         $fileSpec.checksum_info = $null
 	    $fileSpec.Name = $uploadfile
 	    $fileSpec.Source_type = "PUSH"
 	    $file = $cisFile.add($sessionId,$fileSpec)
         $uri = $file.Upload_Endpoint.Uri
         $wc = New-Object -TypeName system.net.webclient
         Write-Verbose "Uploading $fileDirectory\$UploadFile to $($uri.absoluteuri)"
         [void]$wc.UploadFile($uri.absoluteuri,"PUT", "$FileDirectory\$uploadFile")
         $result = $cisFile.validate($sessionId)
         $missingfile = @($result.Missing_Files)
         Write-Verbose "Number of missing files: $($missingFile.Count)"
     }
     while ($missingfile.count)
     $cisUpdateSession.complete($sessionId)
 }
 
 function Remove-CLItem
 {
     [cmdletbinding(SupportsShouldProcess=$true,ConfirmImpact='High')]
     param([Parameter(Mandatory=$true,ValueFromPipelineByPropertyName=$true)][VMware.VimAutomation.Cis.Core.Types.V1.ID]$Id)
     BEGIN
     {
         $cisItem = Get-cisService com.vmware.content.library.Item
     }
     PROCESS
     {
         if($PSCmdlet.ShouldProcess("$((Get-CLItem -Id $Id).Name)", "Remove"))
         {
             $cisItem.delete($Id)
         }
     }
 }
 
 function Export-CLItem
 {
     [cmdletbinding(DefaultParameterSetName='Id')]
     param([Parameter(ParameterSetName='Id',Mandatory=$true,ValueFromPipelineByPropertyName=$true)][VMware.VimAutomation.Cis.Core.Types.V1.ID]$Id,
            [String]$Path = (Get-Location),
            [Parameter(Mandatory=$true,ParameterSetName='Name')][String]$Name,
            [Parameter(ParameterSetName='Name')]$ContentLibrary)
     BEGIN
     {
         if ($PSCmdlet.ParameterSetName -eq 'Name')
         {
             $params = @{} + $PSBoundParameters
             $params.Remove('Path')
             Write-Verbose "Params: $($params.keys)"
             return Get-CLItem @params | Export-CLItem -Path $Path
         }
         $cisDownload = Get-cisService com.vmware.content.library.Item.download_session
         $cisDownloadSessionFile = Get-CisService com.vmware.content.library.item.downloadsession.file
         $wc = New-Object -TypeName system.net.webclient
     }
     PROCESS
     {
         if ($PSCmdlet.ParameterSetName -eq 'Name')
         {
             Write-Verbose "Breaking"
             break
         }
         $DownloadSessionModel = $cisDownload.help.create.create_spec.CreateExample()
         $DownloadSessionModel.library_item_id = $Id
         $Item = Get-CLItem -Id $Id
         $ExportPath = "$Path\$($Item.Name)"
         if (!(Test-Path -Path $ExportPath))
         {
             $null = New-Item -ItemType Directory -Path $ExportPath -Force
         }
         $downloadSessionId = $cisDownload.create([guid]::NewGuid().guid,$downloadSessionModel)
         foreach ($downloadInfo in $cisDownloadSessionFile.list($downloadSessionId))
         {
             $downloadInfo = $cisDownloadSessionFile.prepare($downloadSessionId, $downloadInfo.Name,'HTTPS')
             Write-Verbose "Preparing file $($downloadInfo.Name) for download."
             while ($cisDownloadSessionFile.get($downloadSessionId, $downloadInfo.Name).Status -ne 'PREPARED')
             {
                 Start-Sleep 1
             }
             $uri = $cisDownloadSessionFile.get($downloadSessionId, $downloadInfo.Name).Download_EndPoint.uri
             Write-Verbose "Downloading $($downloadinfo.Name) from $uri to $ExportPath."
             [void]$wc.DownloadFile($uri, "$ExportPath\$($downloadInfo.Name)")
         }
         $cisDownload.delete($downloadSessionId)
     }
 
 }
`

