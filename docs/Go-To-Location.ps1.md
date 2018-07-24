---
Author: d3m80l
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 4398
Published Date: 2014-08-16t19
Archived Date: 2017-05-23t23
---

# go to location - 

## Description

remember locations under defined labels and navigate back to those locations with the label.

## Comments



## Usage



## TODO



## script

`go-tolocation`

## Code

`#
 #
 
 .SYNOPSIS  
     Allows to remember locations by defining them under a user-defined label.
 
 .DESCRIPTION  
     
 .PARAMETER Label
 	Name of label which should be used to identify the stored location.
 
 .PARAMETER Remember
 	Switch informing that the location should be stored under provided label.
 
 .PARAMETER Delete
 	Delete the stored label.
 
 .PARAMETER Info
 	Get information about stored label.
 
 .PARAMETER Location
 	By default this is the current working directory ($pwd).
 	Information which should be stored.
 
 .PARAMETER Confirm
 	Confirm label delete without asking the user.
 
 .NOTES  
     Author         : Marcin Dembowski [myname.mysurname(at)gmail.com]
     Website        : http://marcindembowski.wordpress.com/
     Prerequisite   : PowerShell V3
     License        : Use it as you wish. It would be nice if you will mention this script in your work ;)
 
 .LINK  
     http://marcindembowski.wordpress.com/
 
 .EXAMPLE  
     PS> Go-ToLocation
 
 
     Label          Location             CreatedOn
     -------------- -------------------- ----------------------------
     projects       H:\marcin\projekty   2013-08-12T19:20:16.5052452Z
 
 
     Lists all defined labels, their location and information when the label was created in UTC time.
 
 .EXAMPLE  
     PS> Go-ToLocation projects
 
     Navigates to location defined under the label 'projects'.
 
 .EXAMPLE
     H:\projects> Go-ToLocation -Remember
 
     Label          Location             CreatedOn
     -------------- -------------------- ----------------------------
     projects       H:\projects          2013-08-12T19:30:14.3041231Z
 
     Remembers the current location using current folder name as label name.
     In this case the label will be 'projects'
 
 .EXAMPLE
     PS> Go-ToLocation -Remember mylabel
 
     Remembers the current location under label 'mylabel'
 
 .EXAMPLE  
     PS> Go-ToLocation projects -Delete
 
     Removes the location remembered under 'projects' label.
 
 .EXAMPLE
     PS> Go-ToLocation projects -Info
 
     Label          Location             CreatedOn
     -------------- -------------------- ----------------------------
     projects       H:\projects          2013-08-12T19:30:14.3041231Z
 
 	Returns information about stored label if exists.
 	
 .EXAMPLE
     PS> Go-ToLocation -Location c:\user\myname -Remember home
 
     Label          Location             CreatedOn
     -------------- -------------------- ----------------------------
     myname         C:\user\myname       2013-08-12T19:15:13.4051132Z
 
     Remembers a location provided by parameter under a specified label.
 #>
 
 function Go-ToLocation
 {
 	param
 	(
 		[string]
 		$Label,
 		
 		[switch]
 		$Remember,
 		
 		[switch]
 		$Delete,
 
         [switch]
         $Info,
 
         [string]
         $Location = $pwd,
 
         [switch]
         $Confirm
 	)
 	
 	$labelsPath = "$env:APPDATA\Go-ToLocation";
 	if ((Test-Path $labelsPath) -ne $true) 
 	{
 		(mkdir $labelsPath) 2>&1 | Out-Null
 	}
 	
 	$labelsFile = "$labelsPath\labels.dat"
 	if (-not (Test-Path $labelsFile))
 	{
 		"Label;Location;CreatedOn" | Out-File -FilePath $labelsFile -Encoding UTF8
 	}
 	
 	if (-not (Test-Path $labelsFile))
 	{
 		Write-Error "Cannot process, because the location file '$labelsFile' could not be created."
 		return
 	}
 
 	if ($Remember) 
 	{
         if ([string]::IsNullOrWhiteSpace($Label))
         {
             $Label = Split-Path -leaf -path (Get-Location)
         }
 
 		$existingItem = Import-Csv -Delimiter ';' $labelsFile | ? { $_.Label -eq $Label } | Select-Object Location -First 1;
 		
 		if ($existingItem -ne $null) 
 		{
 			Write-Error "There is already something under '$Label' which points to: '$($existingItem.Location)'"
 			return;
 		}
 		
 		"$Label;$Location;$([datetime]::UtcNow.ToString('o'))" | Out-File -Append -FilePath $labelsFile -Encoding UTF8
 		$Info = $true
 	}
 	
 	if ($Delete) 
 	{
         if ([string]::IsNullOrWhiteSpace($Label))
         {
             $Label = Split-Path -leaf -path (Get-Location)
         }
 
         $existingItem = Import-Csv -Delimiter ';' $labelsFile | ? { $_.Label -eq $Label } | Select-Object Location -First 1;
         if ($existingItem -eq $null)
         {
             return;
         }
 
         if (-not $Confirm) 
         {
             $yes = New-Object System.Management.Automation.Host.ChoiceDescription "&Yes", "Deletes the label '$Label' from list."
             $no = New-Object System.Management.Automation.Host.ChoiceDescription "&No", "Do not delete the label."
             $options = [System.Management.Automation.Host.ChoiceDescription[]]($yes, $no)
             $result = $host.ui.PromptForChoice("Delete label", "Do you want to delete the label '$Label' which points to '$($existingItem.Label)'?", $options, 1) 
             if ($result -eq 1)
             {
                 return;
             }
         }
 		Import-Csv -Delimiter ';' $labelsFile -Encoding UTF8 | ? { $_.Label -ne $Label } | Export-Csv -Delimiter ';' "$labelsFile.tmp" -Encoding UTF8
         copy "$labelsFile.tmp" "$labelsFile" -Force
 		return;
 	}
 
     if ($Info)
     {
         if ([string]::IsNullOrWhiteSpace($Label))
         {
             $Label = Split-Path -leaf -path (Get-Location)
         }
         Import-Csv -Delimiter ';' $labelsFile -Encoding UTF8 | ? { $_.Label -eq $Label } | Select-Object $_ -First 1;
         return;
     }
 	
 	if ([string]::IsNullOrWhiteSpace($Label)) 
 	{
 		Import-Csv -Delimiter ';' $labelsFile;
 		
 		return;
 	}
 	
 	$item = Import-Csv -Delimiter ';' $labelsFile | Where-Object { $_.Label -eq $Label } | Select-Object Location -First 1;
 	
 	if ($item -eq $null)
 	{
 		Write-Error "The provided item is not available";
 		return;
 	}
 	
 	cd $item.Location
 }
 
 Set-Alias -Name go Go-ToLocation -Force
 Export-ModuleMember Go-ToLocation
 Export-ModuleMember -Alias go
`

