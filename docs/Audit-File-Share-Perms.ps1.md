---
Author: grolo
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3398
Published Date: 2012-05-04t06
Archived Date: 2016-10-11t15
---

# audit file share perms - 

## Description

produces html report of the ntfs & share permissions for each share on a system. administrative shares such as admin$ are excluded.

## Comments



## Usage



## TODO



## function

`get-sharepermissions`

## Code

`#
 #
 Function Get-SharePermissions($ShareName){
     $Share = Get-WmiObject win32_LogicalShareSecuritySetting -Filter "name='$ShareName'"
     if($Share){
         $obj = @()
         $ACLS = $Share.GetSecurityDescriptor().Descriptor.DACL
         foreach($ACL in $ACLS){
             $User = $ACL.Trustee.Name
             if(!($user)){$user = $ACL.Trustee.SID}
             $Domain = $ACL.Trustee.Domain
             switch($ACL.AccessMask)
             {
                 2032127 {$Perm = "Full Control"}
                 1245631 {$Perm = "Change"}
                 1179817 {$Perm = "Read"}
             }
             $obj = $obj + "$Domain\$user  $Perm<br>"
         }
     }
     if(!($Share)){$obj = " ERROR: cannot enumerate share permissions. "}
     Return $obj
 
 Function Get-NTFSOwner($Path){
     $ACL = Get-Acl -Path $Path
     $a = $ACL.Owner.ToString()
     Return $a
 
 Function Get-NTFSPerms($Path){
     $ACL = Get-Acl -Path $Path
     $obj = @()
     foreach($a in $ACL.Access){
         $aA = $a.FileSystemRights
         $aB = $a.AccessControlType
         $aC = $a.IdentityReference
         $aD = $a.IsInherited
         $aE = $a.InheritanceFlags
         $aF = $a.PropagationFlags
         $obj = $obj + "$aC | $aB | $aA | $aD | $aE | $aF <br>"
     }
     Return $obj
 
 Function Get-AllShares{
     $a = Get-WmiObject win32_share -Filter "type=0"
     Return $a
 
 $z = "<!DOCTYPE html PUBLIC `"-//W3C//DTD XHTML 1.0 Strict//EN`"  `"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd`">"
 $z = $z + "<html xmlns=`"http://www.w3.org/1999/xhtml`">"
 $z = "<head><style>"
 $z = $z + "TABLE{border-width: 2px;border-style: solid;border-color: black;border-collapse: collapse;}"
 $z = $z + "TH{border-width: 2px;padding: 4px;border-style: solid;border-color: black;background-color:lightblue;text-align:left;font-size:14px}"
 $z = $z + "TD{border-width: 1px;padding: 4px;border-style: solid;border-color: black;font-size:12px}"
 $z = $z + "</style></head><body>"
 $z = $z + "<H4>File Share Report for $env:COMPUTERNAME</H4>"
 $z = $z + "<table><colgroup><col/><col/><col/><col/><col/><col/></colgroup>"
 $z = $z + "<tr><th>ShareName</th><th>Location</th><th>NTFSPermissions<br>IdentityReference|AccessControlType|FileSystemRights|IsInherited|InheritanceFlags|PropagationFlags</th><th>NTFSOwner</th><th>SharePermissions</th><th>ShareDescription</th></tr>"
 
 $MainShares = Get-AllShares
 Foreach($MainShare in $MainShares){
     $MainShareName = $MainShare.Name
     $MainLocation = $MainShare.Path
     $MainNTFSPermissions = Get-NTFSPerms -Path $MainLocation
     $MainNTFSOwner = Get-NTFSOwner -Path $MainLocation
     $MainSharePermissions = Get-SharePermissions -ShareName $MainShareName
     $MainShareDescription = $MainShare.Description
     
     $z = $z + "<tr><td>$MainShareName</td><td>$MainLocation</td><td>$MainNTFSPermissions</td><td>$MainNTFSOwner</td><td>$MainSharePermissions</td><td>$MainShareDescription</td></tr>"
 }
 $z = $z + "</table></body></html>"
 $OutFileName = $env:COMPUTERNAME + "ShareReport.html"
 Out-File -FilePath .\$OutFileName -InputObject $z -Encoding ASCII
 $OutFileItem = Get-Item -Path .\$OutFileName
 Write-Host " Report available here: $OutFileItem" -Foregroundcolor Yellow
 Exit
`

