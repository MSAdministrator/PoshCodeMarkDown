---
Author: hughs
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3014
Published Date: 2011-10-19t11
Archived Date: 2011-10-23t11
---

# deploy file &amp; shortcut - 

## Description

deploys a file to a list of computers pulled from ad and creates a shortcut on each users desktop.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Clear-Host
 {
 	{
 		{
 			IF (Test-Path "\\$Computer\C$\Users\Public\Training")
 				{
 					Copy-Item "C:\Users\Public\Training\HOWTO_VPN.exe" "\\$Computer\C$\Users\Public\Training"
 				}
 			ELSE 
 				{
 					mkdir -Path "\\$Computer\C$\Users\Public\Training"
 					Copy-Item "C:\Users\Public\Training\HOWTO_VPN.exe" "\\$Computer\C$\Users\Public\Training"
 				}
 			FOREACH ($User in $Users)
 					{
 						{
 							{
 									{
 										$lnk = $wshshell.CreateShortcut("\\$Computer\C$\Users\$User\Desktop\Training.lnk")
 										$lnk.TargetPath = "C:\Users\Public\Training"
 										$lnk.Save()
 										Write-Host "$Computer $User DONE"
 									}
 								ELSE
 									{
 									Write-Host "$Computer $User NOPE"
 									}
 							}
 						}
 					}
 				}
 	ELSE
 		{ 
 			IF (Test-Path "\\$Computer\C$\Documents and Settings\All Users\Training")
 				{
 					Copy-Item "C:\Users\Public\Training\HOWTO_VPN.exe" "\\$Computer\C$\Documents and Settings\All Users\Training"
 				}
 			ELSE
 				{
 					mkdir -Path "\\$Computer\C$\Documents and Settings\All Users\Training"
 					Copy-Item "C:\Users\Public\Training\HOWTO_VPN.exe" "\\$Computer\C$\Documents and Settings\All Users\Training"				
 				}
 			$Users = Get-ChildItem "\\$Computer\C$\Documents and Settings"
 			FOREACH ($User in $Users)
 				{
 					IF ("Administrator" -ne $User)
 					{
 						IF (Test-Path "\\$Computer\C$\Users\$User\Desktop")
 							{
 								$lnk = $wshshell.CreateShortcut("\\$Computer\C$\Users\$User\Desktop\Training.lnk")
 								$lnk.TargetPath = "C:\Users\Public\Training"
 								$lnk.Save()
 							Write-Host "$Computer $User DONE"
 							}
 						ELSE
 							{
 							Write-Host "$Computer $User NOPE"
 							}
 					}
 				}
 			}
 		}
 }
`

