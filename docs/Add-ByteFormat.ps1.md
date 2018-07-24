---
Author: bartekb
Publisher: 
Copyright: 
Email: 
Version: 913.00
Encoding: ascii
License: cc0
PoshCode ID: 3393
Published Date: 2012-05-02t15
Archived Date: 2012-05-10t14
---

# add-byteformat - 

## Description

with help of this function you will be able to force nice display of numeric data.

## Comments



## Usage



## TODO



## function

`add-byteformat`

## Code

`#
 #
 function Add-ByteFormat {
 
 <#
     .Synopsis
         Function to make display of custom properties more human-readable.
 
     .Description
         With help of this function you will be able to force nice display of numeric data.
         It's using best possible unit of measure for *bytes sizes.
         If input object is PSCustomObject it will just modify it's ToString() method
         If it's any other type - it will try to remove property, re-add and modify ToString() method.
         Tested both on v2 and v3, works fine in both cases.
 
     .Example
         Get-ChildItem | Add-ByteFormat -Property Length
         
         Output:
                     Directory: C:\temp\PowerShell\vug\ShowUI
 
 
         Mode                LastWriteTime     Length Name                                                                                                                        
         ----                -------------     ------ ----                                                                                                                        
         -a---        26/04/2012     09:36  534.00  B 0_GetProxy.ps1                                                                                                              
         -a---        26/04/2012     11:12  266.00  B 1_StackPanel.ps1                                                                                                            
         -a---        25/04/2012     19:44  752.00  B 2_SimpleUI.ps1                                                                                                              
         -a---        25/04/2012     19:43  913.00  B 3_Przezroczyste.ps1                                                                                                         
         -a---        25/04/2012     19:41    1.28 KB 4_ADLastName.ps1                                                                                                            
         -a---        25/04/2012     19:42    1.38 KB 5_ADLastName.ps1                                                                                                            
         -a---        25/04/2012     19:41  343.00  B 6_RandomFont.ps1                                                                                                            
         -a---        25/04/2012     19:54    1.42 KB 7_ADLastName.ps1                                                                                                            
         -a---        26/04/2012     11:17  624.00  B 8_Show-Command.ps1                                                                                                          
         -a---        26/04/2012     10:28    8.01 KB 9_Show-Picture.ps1                                                                                                          
         -a---        25/04/2012     19:27   21.29 KB logo.png                                                                                                                    
         -a---        15/10/2011     10:34    1.14 MB Normal.png                                                                                                                  
         -a---        25/05/2011     11:28  222.93 KB Powershell-V2-1280x1024.jpg                                                                                                 
         -a---        25/05/2011     11:26  232.69 KB PowerShell1024x768[1].jpg                                                                                                   
         -a---        09/01/2011     21:59   80.36 KB powershellpl.png                                                                                                            
         -a---        26/04/2012     07:18    5.27 KB Sm-.csv                                                                                                                     
         -a---        26/04/2012     11:18    8.08 KB WAR.csv                                                                                                                     
 
 Changes format of the Length property for files in current directory.
 As a side effect in v3 it will show folders as having 1B size.
 
     .Example
         Get-WmiObject -Class Win32_LogicalDisk | Add-ByteFormat -Property Size, FreeSpace -DecimalPoint 0
         
         Output:
     DeviceID     : C:
     DriveType    : 3
     ProviderName : 
     FreeSpace    : 9 GB
     Size         : 298 GB
     VolumeName   : PXL
 
     Will display disk information using WMI in N0 format.
 
     .Example
         Import-Csv Foo.csv | Add-ByteFormat -Property HDSize, RAM | Format-Table -AutoSize
 
         Output:
         Name     HDSize     RAM
         ----     ------     ---
         One   120.50 GB 1.47 GB
         Two    58.10 GB 3.47 GB
         Three  88.01 GB 1.21 GB
 
         This command is wash & go for data imported from CSV that contains numeric:
         -- numeric are no longer strings
         -- numeric are displayed in nice(r) fasion.
 
     .Link
         http://becomelotr.wordpress.com/2012/05/03/more-on-output/
 
 
 #>
 
 
 [CmdletBinding()]
 param (
 
     [Parameter(
         Mandatory = $true
     )]
     [string[]]$Property,
 
     [Alias('DP')]
     [int]$DecimalPoint = 2,
 
     [Parameter(
         ValueFromPipeline = $true,
         Mandatory = $true
     )]
     [Object]$InputObject
 )
 
 begin {
     $MethodOptions = @{
         Name = 'ToString'
         MemberType = 'ScriptMethod'
         PassThru = $true
         Force = $true
         Value = [ScriptBlock]::Create(@"
             "{0:N$DecimalPoint} {1}" -f @(
                 switch -Regex ([math]::Log(`$this,1024)) {
                     ^0 {
                         (`$this / 1), ' B'
                     }
                     ^1 {
                         (`$this / 1KB), 'KB'
                     }
                     ^2 {
                         (`$this / 1MB), 'MB'
                     }
                     ^3 {
                         (`$this / 1GB), 'GB'
                     }
                     ^4 {
                         (`$this / 1TB), 'TB'
                     }
                     default {
                         (`$this / 1PB), 'PB'
                     }
                 }
             )
 "@
         )
         
     }
 }
 
 process {
     
     foreach ($Prop in $Property) {
         $SelectedProperty = $InputObject.$Prop
         if (!$SelectedProperty) {
             Write-Verbose "No such property: $Prop"
             continue
             
         }
 
         if ( ! ([double]$InputObject.$Prop) ) {
             Write-Verbose "Can't be casted into double: $Prop"
             continue
         }
 
 
         if ($SelectedProperty -is [System.String]) {
             [Int64]$InputObject.$Prop = $SelectedProperty
         }
 
         
         if (!$InputObject.PSTypeNames.Contains('System.Management.Automation.PSCustomObject')) {
             try {
                 $Member = @{
                     MemberType = 'NoteProperty'
                     Name = $Prop
                     Force = $true
                     Value = $SelectedProperty
                 }
                 $InputObject | Add-Member @Member
             } catch {
                 Write-Verbose "Not able to replace property on this type: $($InputObject.GetType().FullName)"
                 continue
             }
         }
         
         $InputObject.$Prop = $InputObject.$Prop | Add-Member @MethodOptions
 
     }
     $InputObject
 }
 }
 
 
 ([PSCustomObject]@{
     Name = 'Anna'
     HDSize = 21313123
     RAM = 123GB
     Foo = '12313213123'
 }),([PSCustomObject]@{
     Name = 'Ewa'
     HDSize = 213112313123
     RAM = 1234MB
     Foo = '12313213123'
 }),([PSCustomObject]@{
     Name = 'Adam'
     HDSize = 21313
     RAM = 530PB
     Foo = '1231321113123'
 }) | Add-ByteFormat -Property HDSize, RAM, Foo -Verbose
 
 @'
 Name,HDSize,RAM,Foo
 Anna,21313123,12313213123,12313213123123
 Ewa,123123123,123123123123,0
 Adam,1231312312313,1231313213123123,12313123123123
 '@ | ConvertFrom-Csv | Add-ByteFormat -Property HDSize, RAM, Foo -Verbose
`

