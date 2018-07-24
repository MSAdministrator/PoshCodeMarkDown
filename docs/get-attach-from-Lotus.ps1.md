---
Author: chris weislak
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3741
Published Date: 2014-11-02t21
Archived Date: 2014-11-13t06
---

# get attach. from lotus - 

## Description

get-attachments from lotus notes mail database. it will go through all the email for “today” and download it to a folder you specify.

## Comments



## Usage



## TODO



## function

`get-attachments`

## Code

`#
 #
 Param (
 	[Parameter(ValueFromPipelineByPropertyName=$True,
 		HelpMessage='Lotus Domino Server')]
 		$ServerName,
 	[Parameter(ValueFromPipelineByPropertyName=$True,
 		HelpMessage='Lotus Mail Database')]
 		$Database,
 	[Parameter(ValueFromPipelineByPropertyName=$True,
 		HelpMessage='View to Select')]
 		$View,
 	[Parameter(ValueFromPipelineByPropertyName=$True,
 		HelpMessage='Folder to save Attachments to.')]
 		$Folder
 )
 Function Get-Attachments{
 	[CmdletBinding()]
 	Param (
 		[Parameter(ValueFromPipelineByPropertyName=$True,
 			HelpMessage='Lotus Domino Server')]
 			$ServerName,
 		[Parameter(ValueFromPipelineByPropertyName=$True,
 			HelpMessage='Lotus Mail Database')]
 			$Database,
 		[Parameter(ValueFromPipelineByPropertyName=$True,
 			HelpMessage='View to Select')]
 			$View,
 		[Parameter(ValueFromPipelineByPropertyName=$True,
 			HelpMessage='Folder to save Attachments to.')]
 			$Folder
 	)
 	Begin {
 		$NotesSession = New-Object -ComObject Lotus.NotesSession
 		$NotesSession.Initialize()
 	}
 	Process {
 		$NotesDatabase = $NotesSession.GetDatabase( $ServerName, $Database, 1 )
 		$AllViews = $NotesDatabase.Views | Select-Object -ExpandProperty Name
 		$dbview = $AllViews | Select-String -Pattern $View
 		Foreach ($nview in $dbview) {
 			$view = $NotesDatabase.GetView($nview)
 			$viewnav = $view.CreateViewNav()
 			$docs = $viewnav.GetFirstDocument()
 			while ($docs -ne $null){
 				$document = $docs.Document
 				$docdate = ($document.Created).ToShortDateString()
 				$date = (Get-Date).ToShortDateString()
 				if ($docdate -eq $date){
 					if ($document.HasEmbedded){
 						foreach ($itm in $document.Items){
 							if ($itm.type -eq 1084){
 								$attach = $document.GetItemValue($itm.Name)
 								#$attach = $document.GetItemValue('$File')
 								$attachment = $document.GetAttachment($attach)
 								$atname = $attachment.Source
 								if (Get-Item $Folder\$atname){
 									$extra = (Get-Date -UFormat "%H.%M.%S").toString()
 										$attachment.ExtractFile("$Folder\$extra_$atname")
 								}
 								$attachment.ExtractFile("$Folder\$atname")
 							}
 						}
 					}
 				}
 				$docs = $viewnav.GetNextDocument($docs)
 			}
 		}
 	}
 	End {}
 }
 Get-Attachments -ServerName $ServerName -Database $Database -View $View -Folder $Folder
`

