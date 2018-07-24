---
Author: dollarunderscore
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5337
Published Date: 2015-07-30t14
Archived Date: 2015-01-31t07
---

# get local groups/members - 

## Description

advanced function to get local members. some of text-fetching going on, but it will hopefully work on your machine aswell.

## Comments



## Usage



## TODO



## function

`get-localgroup`

## Code

`#
 #
 #========================================================================
 #========================================================================
 
 function Get-LocalGroup
 {
     <#
     .SYNOPSIS
     Retrieves local groups on a server
 
     .DESCRIPTION
     This cmdlet will retrieve all local groups found a server
 
     .EXAMPLE
     Get-LocalGroup -ComputerName 'MyMachine'
 
     .PARAMETER ComputerName
     The computer or IP-adress to connect to
 
     #>
 
     [CmdletBinding()]
     param(
       [Parameter(Mandatory=$false, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
       [Alias('hostname')]
       [string] $ComputerName = $env:COMPUTERNAME)
 
 
     BEGIN { }
 
     PROCESS {
         $LocalGroupsWMI=Get-WmiObject -ComputerName $ComputerName -Query "Select * from Win32_Group  Where LocalAccount = $true"
         foreach ($LocalGroup in $LocalGroupsWMI) {
             $returnObj = New-Object System.Object
             $returnObj | Add-Member -Type NoteProperty -Name ComputerName -Value $Localgroup.Domain
             $returnObj | Add-Member -Type NoteProperty -Name GroupName -Value $Localgroup.Name
             $returnObj | Add-Member -Type NoteProperty -Name SID -Value $Localgroup.SID
 
             Write-Output $returnObj
         }
     }
 
     END { }
 }
 
 function Get-LocalGroupMember
 {
     <#
     .SYNOPSIS
     Retrieves members of a local group on a server
 
     .DESCRIPTION
     This cmdlet will retrieve all members of the specified local group.
 
     .EXAMPLE
     Get-LocalGroupMember -GroupName Administrators
 
     .EXAMPLE
     Get-LocalGroupMember -ComputerName 'MyMachine' -GroupName Administrators
 
     .PARAMETER ComputerName
     The computer or IP-adress to connect to.
 
     .PARAMETER GroupName
     The name of the local group which members you want to list.
 
     #>
 
     [CmdletBinding()]
     param(
       [Parameter(Mandatory=$false, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
       [Alias('hostname')]
       [string] $ComputerName = $env:COMPUTERNAME,
       [Parameter(Mandatory=$True, ValueFromPipelineByPropertyName=$true)]
       [Alias('Group')]
       [string] $GroupName)
 
     BEGIN { }
 
     PROCESS {
         $LocalGroupMembersWMI=Get-WmiObject -ComputerName $ComputerName -Query "SELECT * FROM Win32_GroupUser WHERE GroupComponent=`"Win32_Group.Domain='$ComputerName',Name='$GroupName'`""
 
         foreach ($LocalGroupWMI in $LocalGroupMembersWMI) {
             $data = $LocalGroupWMI.PartComponent -split "\," 
             $domain = ($data[0] -split "=")[1] -replace """",""
             $name = ($data[1] -split "=")[1] -replace """",""
             $type = ($data[0] -split "_|\.")[1]  -replace "Account"
             $arr += ("$domain\$name").Replace("""","") 
 
             $returnObj = New-Object System.Object
             $returnObj | Add-Member -Type NoteProperty -Name Domain -Value $domain
             $returnObj | Add-Member -Type NoteProperty -Name GroupName -Value $GroupName
             $returnObj | Add-Member -Type NoteProperty -Name Type -Value $type
             $returnObj | Add-Member -Type NoteProperty -Name Name -Value $name
 
             Write-Output $returnObj
         }
     }
 
     END { }
 }
`

