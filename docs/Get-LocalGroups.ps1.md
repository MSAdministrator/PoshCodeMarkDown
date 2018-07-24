---
Author: tojo2000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1268
Published Date: 2010-08-12t21
Archived Date: 2017-02-03t16
---

# get-localgroups - 

## Description

i import this as a module because it depends on the other two functions.  you could easily dot-source it as well.  requires v2 ctp3 or higher to get the help documentation to work.

## Comments



## Usage



## TODO



## function

`add-noteproperty`

## Code

`#
 #
 function Add-NoteProperty {
   <#
 .Synopsis
   Adds a NoteProperty member to an object.
 .Description
   This function makes adding a property a lot easier than Add-Member, assuming
   you want to add a NoteProperty, which I find is true about 90% of the time.
 .Parameter object
   The object to add the property to.
 .Parameter name
   The name of the new property.
 .Parameter value
   The object to add as the property.
 .Example
   PS> $custom_obj = New-Object PSObject
   PS> Add-NoteProperty $custom_obj Name 'Custom'
   PS> Add-NoteProperty $custom_obj Value 42
 .Example
   PS> $bunch_of_objects | Add-NoteProperty -name 'meaning_of_life' -value 42
 .Notes
   NAME:      Add-NoteProperty
   AUTHOR:    Tim Johnson <tojo2000@tojo200.com>
   FILE:      LocalGroups.psm1
 #>
   param([Parameter(Mandatory = $true,
   ValueFromPipeline = $true,
   Position = 0)]
   $object,
   [Parameter(Mandatory = $true,
   Position = 1)]
   [string]$name,
   [Parameter(Mandatory = $true,
   Position = 2)]
   $value)
 
   BEGIN{}
 
   PROCESS{
     $object | Add-Member -MemberType NoteProperty `
     -Name $name `
     -Value $value
   }
 
   END{}
 }
 
 
 function Get-COMProperty{
 <#
 .Synopsis
   Gets a property of a __ComObject object.
 .Description
   This function calls the InvokeMember static method of the class to get 
   properties that aren't directly exposed to PowerShell, such as local group
   members found by calling Members() on a DirectoryServices group object.
 .Parameter com_object
   The object to retrieve the property from.
 .Parameter property_name
   The name of the property.
 .Example
   PS> [adsi]$computer = 'WinNT://servername'
   PS> $groups = $groups = $computer.psbase.children |
   >>    ?{$_.psbase.schemaclassname -eq 'group'}
   PS> $groups[0].Members() | %{Get-COMProperty $_ 'Name'}
 .Notes
   NAME:      Get-COMProperty
   AUTHOR:    Tim Johnson <tojo2000@tojo200.com>
   FILE:      LocalGroups.psm1
 #>
   param([Parameter(Mandatory = $true,
   Position = 0)]
   $com_object,
   [Parameter(Mandatory = $true,
   Position = 1)]
   [string]$property_name)
 
   [string]$property = $com_object.GetType().InvokeMember($property_name, 
   'GetProperty', 
   $null, 
   $com_object, 
   $null)
   Write-Output $property
 }
 
 
 function Get-LocalGroups{
 <#
 .Synopsis
   Gets a list of objects with information about local groups and their members.
 .Description
   This function returns a list of custom PSObjects that are a list of local
   groups on a computer.  Each object has a property called Members that is a
   list of PSObjects representing each member, with Name, Domain, and ADSPath.
 .Parameter computername
   The object to retrieve the property from.
 .Example
   PS> Get-LocalGroups servername
 .Notes
   NAME:      Get-LocalGroups
   AUTHOR:    Tim Johnson <tojo2000@tojo200.com>
   FILE:      LocalGroups.psm1
 #>
   param([Parameter(Mandatory = $true,
   ValueFromPipeline = $true)]
   [string]$computername = $env:computername)
 
   BEGIN{}
 
   PROCESS{
     $output = @()
     [adsi]$computer = "WinNT://$computername"
     $groups = $computer.psbase.children |
     ?{$_.psbase.schemaclassname -eq 'group'}
 
     foreach ($group in $groups) {
       $members = @()
       $grp_obj = New-Object PSObject
       Add-NoteProperty $grp_obj 'Name' $group.Name.ToString()
       Add-NoteProperty $grp_obj 'aDSPath' $group.aDSPath.ToString()
 
       foreach ($user in $group.Members()){
         $usr_obj = New-Object PSObject
         Add-NoteProperty $usr_obj 'aDSPath' (Get-COMProperty $user 'aDSPath')
         Add-NoteProperty $usr_obj 'Name' (Get-COMProperty $user 'Name')
         $path = $usr_obj.aDSPath.split('/')
 
         if ($path.Count -eq 4){
           Add-NoteProperty $usr_obj 'Domain' $path[2]
         }elseif ($path.Count -eq 5) {
           Add-NoteProperty $usr_obj 'Domain' $path[3]
         }else{
           Add-NoteProperty $usr_obj 'Domain' 'Unknown'
         }
         $members += $usr_obj
       }
       $members = @($members | Sort-Object -Property Domain, Name)
       Add-NoteProperty $grp_obj 'Members' $members
       Write-Output $grp_obj
     }
   }
   END{} 
 }
`

