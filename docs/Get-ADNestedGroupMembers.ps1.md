---
Author: guyintheoffice
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6339
Published Date: 2016-05-10t21
Archived Date: 2016-09-08t23
---

# get-adnestedgroupmembers - 

## Description

microsoft provided script, however i can’t download it at work but i can copy and paste.

## Comments



## Usage



## TODO



## function

`get-adnestedgroupmembers`

## Code

`#
 #
 function Get-ADNestedGroupMembers { 
 .EXAMPLE   
 Get-ADNestedGroupMembers "MyGroup" | Export-CSV .\NedstedMembers.csv -NoTypeInformation
 
 .EXAMPLE  
 Get-ADGroup "MyGroup" | Get-ADNestedGroupMembers | ft -autosize
             
 .EXAMPLE             
 Get-ADNestedGroupMembers "MyGroup" -indent
  #>
 
 param ( 
 [Parameter(ValuefromPipeline=$true,mandatory=$true)][String] $GroupName, 
 [int] $nesting = -1, 
 [int]$circular = $null, 
 [switch]$indent 
 ) 
     function indent  
     { 
     Param($list) 
         foreach($line in $list) 
         { 
         $space = $null 
          
             for ($i=0;$i -lt $line.nesting;$i++) 
             { 
             $space += "    " 
             } 
             $line.name = "$space" + "$($line.name)"
         } 
       return $List 
     } 
      
 $modules = get-module | select -expand name
     if ($modules -contains "ActiveDirectory") 
     { 
         $table = $null 
         $nestedmembers = $null 
         $adgroupname = $null     
         $nesting++   
         $ADGroupname = get-adgroup $groupname -properties memberof,members 
         $memberof = $adgroupname | select -expand memberof 
         write-verbose "Checking group: $($adgroupname.name)" 
         if ($adgroupname) 
         {  
             if ($circular) 
             { 
                 $nestedMembers = Get-ADGroupMember -Identity $GroupName -recursive 
                 $circular = $null 
             } 
             else 
             { 
                 $nestedMembers = Get-ADGroupMember -Identity $GroupName | sort objectclass -Descending
                 if (!($nestedmembers))
                 {
                     $unknown = $ADGroupname | select -expand members
                     if ($unknown)
                     {
                         $nestedmembers=@()
                         foreach ($member in $unknown)
                         {
                         $nestedmembers += get-adobject $member
                         }
                     }
 
                 }
             } 
  
             foreach ($nestedmember in $nestedmembers) 
             { 
                 $Props = @{Type=$nestedmember.objectclass;Name=$nestedmember.name;DisplayName="";ParentGroup=$ADgroupname.name;Enabled="";Nesting=$nesting;DN=$nestedmember.distinguishedname;Comment=""} 
                  
                 if ($nestedmember.objectclass -eq "user") 
                 { 
                     $nestedADMember = get-aduser $nestedmember -properties enabled,displayname 
                     $table = new-object psobject -property $props 
                     $table.enabled = $nestedadmember.enabled
                     $table.name = $nestedadmember.samaccountname
                     $table.displayname = $nestedadmember.displayname
                     if ($indent) 
                     { 
                     indent $table | select @{N="Name";E={"$($_.name)  ($($_.displayname))"}}
                     } 
                     else 
                     { 
                     $table | select type,name,displayname,parentgroup,nesting,enabled,dn,comment 
                     } 
                 } 
                 elseif ($nestedmember.objectclass -eq "group") 
                 {  
                     $table = new-object psobject -Property $props 
                      
                     if ($memberof -contains $nestedmember.distinguishedname) 
                     { 
                         $table.comment ="Circular membership" 
                         $circular = 1 
                     } 
                     if ($indent) 
                     { 
                     indent $table | select name,comment | %{
 						
 						if ($_.comment -ne "")
 						{
 						[console]::foregroundcolor = "red"
 						write-output "$($_.name) (Circular Membership)"
 						[console]::ResetColor()
 						}
 						else
 						{
 						[console]::foregroundcolor = "yellow"
 						write-output "$($_.name)"
 						[console]::ResetColor()
 						}
                     }
 					}
                     else 
                     { 
                     $table | select type,name,displayname,parentgroup,nesting,enabled,dn,comment 
                     } 
                     if ($indent) 
                     { 
                        Get-ADNestedGroupMembers -GroupName $nestedmember.distinguishedName -nesting $nesting -circular $circular -indent 
                     } 
                     else  
                     { 
                        Get-ADNestedGroupMembers -GroupName $nestedmember.distinguishedName -nesting $nesting -circular $circular 
                     } 
               	                  
                } 
                 else 
                 { 
                     
                     if ($nestedmember)
                     {
                         $table = new-object psobject -property $props
                         if ($indent) 
                         { 
     	                    indent $table | select name 
                         } 
                         else 
                         { 
                         $table | select type,name,displayname,parentgroup,nesting,enabled,dn,comment    
                         } 
                      }
                 } 
               
             } 
          } 
     } 
     else {Write-Warning "Active Directory module is not loaded"}        
 }
`

