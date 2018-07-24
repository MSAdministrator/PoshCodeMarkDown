---
Author: brodobrey
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3629
Published Date: 2012-09-07t06
Archived Date: 2012-09-15t03
---

# tst.ps1 - 

## Description

tst.ps1

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 
 
 
 
 function WaitForJobToFinish([string]$SolutionFileName)
 {
     $JobName = "*solution-deployment*$SolutionFileName*"
     $job = Get-SPTimerJob | ?{ $_.Name -like $JobName }
     if ($job -eq $null)
     {
         Write-Host 'Timer job not found'
     }
     else
     {
         $JobFullName = $job.Name
         Write-Host -NoNewLine "Waiting to finish job $JobFullName"
       
         while ((Get-SPTimerJob $JobFullName) -ne $null)
         {
             Write-Host -NoNewLine .
             Start-Sleep -Seconds 2
         }
         Write-Host  "Finished waiting for job.."
     }
 }
 
 
 
 $snapin = Get-PSSnapin | Where-Object {$_.Name -eq 'Microsoft.SharePoint.Powershell'}
  if ($snapin -eq $null) 
  {
   Write-Host "Loading SharePoint Powershell Snap-in"
   Add-PSSnapin "Microsoft.SharePoint.Powershell"
  }
 
 
 $user=[System.Security.Principal.WindowsIdentity]::GetCurrent().Name
 
 Write-Host  "current user = $user"
 Write-Host  "Deploying Solution Started.."
 
 $WSP = Get-SPSolution | Where { 
     ($solution -eq $_.Name) 
 } 
 if($WSP -eq $null) { 
 }
 else
 {
     Write-Host  "De-Activating feature -"$feature
     Disable-SPFeature -identity $feature -Url $targetSiteColURL -Confirm:$false
     Write-Host  "Feature de-activated - "$feature
   
     Write-Host  "Found " $WSP " - Uninstalling."
     Uninstall-SPSolution -Identity $solution -Confirm:$false
     WaitForJobToFinish($solution)
     Write-Host  $WSP "Uninstalled."
   
     Write-Host  "removing Solution -" $solution
     Remove-SPSolution -Identity $solution -Confirm:$false
     WaitForJobToFinish($solution)
     Write-Host  "removed Solution -" $solution
   
     Write-Host  "Deleting target site -" $targetUrl
     Remove-SPWEB -identity $targetUrl -Confirm:$false
     Write-Host  "Deleted target site - " $targetUrl
 
   
 }
 
 
 
 Write-Host  "Adding Solution"
 Add-SPSolution -LiteralPath $path -Confirm:$false
 Write-Host  "Added Solution"
 
 Write-Host  "Installing Solution"
 Install-SPSolution -Identity $solution -GACDeployment -Confirm:$false
 WaitForJobToFinish($solution)
 Write-Host  "Installed Solution"
 
 Write-Host  "Activating feature - "$feature
 Enable-SPFeature -identity $feature -Url $targetWebAppURL -Confirm:$false
 Write-Host  "Feature activated - " $feature
 
 $site= new-Object Microsoft.SharePoint.SPSite($targetWebAppUrl)
 #$loc= [System.Int32]::Parse(1033)
 #$templates= $site.GetWebTemplates($loc)
 $templates= $site.GetWebTemplates($loc)
 $templateId=""
 foreach ($child in $templates)
 {   
     if($child.Title -eq $SiteTitle)
     {
         $templateId= $child.Name
     
     }
 
 }
 $site.Dispose()
 
 
 
 Write-Host  "Creating New site with templateID -" $templateId
 $web = New-SPWeb -Url $targetUrl -Name $SiteName -Description $SiteDesc -AddToTopNav -Confirm:$false
 Write-Host "New Site Created"
 Write-Host "Applying template please wait....."
 $web.ApplyWebTemplate($templateId)
 $web.Dispose()
 Write-Host "Site template applied successfully! Template = $templateId "
 Write-Host -Fore Green "Solution deployment completed"
 Write-Host "To visit the created site Browse - "-Fore Blue $targetUrl
 get-pssession | remove-pssession
 
`

