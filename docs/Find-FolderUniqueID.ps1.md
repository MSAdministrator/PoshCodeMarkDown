---
Author: chad c
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5919
Published Date: 2016-07-06t23
Archived Date: 2016-10-27t12
---

# find-folderuniqueid - 

## Description

finds exchange mailbox folder unique id for use with exchange web service managed api.  at times you may want to work with a folder that is not a well known folder (inbox,sent items, etc).  this will get you the unique id required to bind to that folder.

## Comments



## Usage



## TODO



## function

`find-folderuniqueid`

## Code

`#
 #
 function Find-FolderUniqueID
 {
        
        <#
        .SYNOPSIS
               Find exchange folder unique ID for use in Exchange Web Service Managed API
        
        .EXAMPLE
               Find-FolderUniqueID -MailBox "joeschmoe@exchange.com" -FolderName "Special_Folder"
 
        .EXAMPLE
               Find-FolderUniqueID -Mailbox "joeschmoe@exchange.com" -FolderName "Special_Folder" -ExchangeVersion Exchange2013
 
        .EXAMPLE
               Find-FolderUniqueID -Mailbox "joeschmoe@exchange.com" -FolderName "Special_Folder" -AlternateCreds | Format-List *
 
        .PARAMETER MailBox
               Specify Mailbox you want to connect to.
        
        .PARAMETER FolderName
               Specify folder name to search for.
        
        .PARAMETER AlternateCreds
               Specify alternate credentials.
 
        .PARAMETER ExchangeVersion
               Specify your exchange environment version.
 
         .NOTES
         Requires EWS Managed API libraries
         Requires Exchange Environment
               
         .LINK
             http://www.microsoft.com/en-us/download/details.aspx?id=35371
             https://msdn.microsoft.com/en-us/library/microsoft.exchange.webservices.data.exchangeversion(v=exchg.80).aspx
        #>
        
        [CmdletBinding()]
        [OutputType([psobject])]
        param
        (
               [Parameter(Mandatory = $true,
               ValueFromPipeline = $false)]
               [string]$MailBox,
               [Parameter(Mandatory = $true)]
               [string]$FolderName,
               [Parameter(Mandatory = $false)]
               [switch]$AlternateCreds,
               [Parameter(Mandatory = $false)]
               [ValidateSet('Exchange2007_SP1','Exchange2010','Exchange2010_SP1','Exchange2010_SP2','Exchange2013','Exchange2013_SP1')]
               [string]$ExchangeVersion = 'Exchange2010_SP2'
        )
        
        $ewsPath = "C:\Program Files\Microsoft\Exchange\Web Services\2.0\Microsoft.Exchange.WebServices.dll"
        Add-Type -Path $ewsPath
        $ews = New-Object Microsoft.Exchange.WebServices.Data.ExchangeService -ArgumentList $ExchangeVersion
        
        if ($AlternateCreds)
        {
               $cred = (Get-Credential).GetNetworkCredential()
               $ews.Credentials = New-Object System.Net.NetworkCredential -ArgumentList $cred.UserName, $cred.Password, $cred.Domain
        }
        else
        {
               $ews.UseDefaultCredentials = $true
        }
        $ews.AutodiscoverUrl($MailBox)
        
        $view = New-Object Microsoft.Exchange.WebServices.Data.FolderView(100)
        $view.PropertySet = New-Object Microsoft.Exchange.WebServices.Data.PropertySet([Microsoft.Exchange.Webservices.Data.BasePropertySet]::FirstClassProperties)
        $view.PropertySet.Add([Microsoft.Exchange.Webservices.Data.FolderSchema]::DisplayName)
        $view.Traversal = [Microsoft.Exchange.Webservices.Data.FolderTraversal]::Deep
        
        $folderid = new-object Microsoft.Exchange.WebServices.Data.FolderId([Microsoft.Exchange.WebServices.Data.WellKnownFolderName]::MsgFolderRoot, "$MailBox")
        
        $findResults = $ews.FindFolders($folderid, $view)
        
        
        $output = @()
        
        foreach ($f in $findResults)
        {
               if ($f.DisplayName -match $FolderName)
               {
                      $data = @{
                            Mailbox = $MailBox
                            FolderName = $f.DisplayName
                            FolderID = $f.Id.UniqueId
                      }
                      $output += New-Object -TypeName PSObject -Property $data
                      
               }
        }
        
        $output
 }
`

