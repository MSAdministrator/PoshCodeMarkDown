---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2751
Published Date: 2011-06-25t16
Archived Date: 2011-06-28t06
---

# rollback policy module - 

## Description

this module allows you to get, enable and disable the software rollback settings on your computer or computers. this is useful when working with certain patches and software installations that require the software rollback to be enabled.

## Comments



## Usage



## TODO



## function

`get-rollbackpolicy`

## Code

`#
 #
 Function Get-RollbackPolicy { 
 .SYNOPSIS   
      Retieves the current rollback policy on a local or remote computer 
           
 .DESCRIPTION   
     Retieves the current rollback policy on a local or remote computer 
      
 .PARAMETER Computername 
     The name of the computer or computers to perform the query against 
                 
 .NOTES   
     Name: Get-RollbackPolicy 
     Author: Boe Prox 
     DateCreated: 06/24/2011 
     Links:  
  
 .EXAMPLE 
 Get-RollbackPolicy -Computername DC1 
  
 Rollback                                Computer                                
 --------                                --------                                
 Enabled                                 dc1      
  
 Description 
 ----------- 
 Performs a query against the server, DC1 and returns whether the rollback setting has been enabled  
 or disabled. 
  
 .EXAMPLE 
 Get-RollbackPolicy -Computername DC1,server1,server2 
  
 Rollback                                Computer                                
 --------                                --------                                
 Enabled                                 dc1      
 Disabled                                server1    
 Enabled                                 server2    
  
 Description 
 ----------- 
 Performs a query against the server, DC1 and returns whether the rollback setting has been enabled  
 or disabled. 
 #> 
 [cmdletbinding()] 
 Param ( 
     [parameter()] 
     [string[]]$Computername 
     ) 
 Process { 
     ForEach ($computer in $computername) { 
         Try { 
             Write-Verbose "Making registry connection to remote computer"      
             $registry = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey("LocalMachine",$computer) 
             Write-Verbose "Making registry connection to 'Software' key"  
             If ($registry.GetSubkeyNames() -contains "Software") { 
                 $subkey = $registry.OpenSubKey("Software",$True) 
                 Write-Verbose "Making registry connection to 'Policies' key" 
                 If ($subkey.GetSubKeyNames() -contains "Policies") { 
                     $subkey = $subkey.OpenSubKey("Policies",$True) 
                     Write-Verbose "Making registry connection to 'Microsoft' key" 
                     If ($subkey.GetSubKeyNames() -contains "Microsoft") { 
                         $subkey = $subkey.OpenSubKey("Microsoft",$True) 
                         Write-Verbose "Making registry connection to 'Windows' key" 
                         If ($subkey.GetSubKeyNames() -contains "Windows") { 
                             $subkey = $subkey.OpenSubKey("Windows",$True) 
                             Write-Verbose "Making registry connection to 'Installer' key" 
                             If ($subkey.GetSubKeyNames() -contains "Installer") { 
                                 $subkey = $subkey.OpenSubKey("Installer",$True) 
                                 Write-Verbose "Checking to see if 'DisableRollback' value name exists" 
                                 If ($subkey.GetValueNames() -contains "DisableRollback") { 
                                     Write-Verbose "Retrieving value of 'DisableRollback'" 
                                     $value = $subkey.GetValue('DisableRollback') 
                                     Switch ($value) { 
                                         0 {$hash = @{Computer = $computer;Rollback = "Enabled"}} 
                                         1 {$hash = @{Computer = $computer;Rollback = "Disabled"}} 
                                         Default {$hash = @{Computer = $computer;Rollback = "Unknown"}} 
                                         }                                     
                                     } 
                                 Else { 
                                     Write-Warning "$($computer): Missing 'DisableRollback' value name in registy!" 
                                     $hash = @{Computer = $computer;Rollback = "Missing DisableRollback value name"} 
                                     } 
                                 } 
                             Else { 
                                 Write-Warning "$($computer): Missing 'Installer' key in registy!" 
                                 $hash = @{Computer = $computer;Rollback = "Missing Installer key"} 
                                 } 
                             } 
                         Else { 
                             Write-Warning "$($computer): Missing 'Windows' key in registy!" 
                             $hash = @{Computer = $computer;Rollback = "Missing Windows key"} 
                             } 
                         } 
                     Else { 
                         Write-Warning "$($computer): Missing 'Microsoft' key in registy!" 
                         $hash = @{Computer = $computer;Rollback = "Missing Microsoft key"} 
                         } 
                     } 
                 Else { 
                     Write-Warning "$($computer): Missing 'Policies' key in registy!" 
                     $hash = @{Computer = $computer;Rollback = "Missing Policies key"} 
                     } 
                 } 
             Else { 
                 Write-Warning "$($computer): Missing 'Software' key in registy!" 
                 $hash = @{Computer = $computer;Rollback = "Missing Software key"} 
                 } 
             } 
         Catch { 
             Write-Warning "$($computer): $($Error[0])" 
             $hash = @{Computer = $computer;Rollback = ($error[0].exception.innerexception.message -split "`n")[0]} 
             } 
         Finally { 
             $object = New-Object PSObject -Property $hash 
             $object.PSTypeNames.Insert(0,'RollbackStatus') 
             Write-Output $object         
             } 
         } 
     } 
 } 
  
 Function Disable-RollbackPolicy { 
 .SYNOPSIS   
      Disables the rollback policy on a local or remote computer 
      
 .DESCRIPTION   
     Disables the rollback policy on a local or remote computer 
      
 .PARAMETER Computername 
     The name of the computer or computers to disable the rollback policy on 
  
 .PARAMETER Passthru 
     Displays the returned object after disabling the policy. 
      
 .NOTES   
     Name: Disable-RollbackPolicy 
     Author: Boe Prox 
     DateCreated: 06/24/2011 
     Links:  
  
 .EXAMPLE 
 Disable-RollbackPolicy -Computername DC1 -Passthru 
  
 Rollback                                Computer                                
 --------                                --------                                
 Disabled                                DC1     
  
 Description 
 ----------- 
 This disables the rollback policy on DC1 and returns the object showing that the policy has  
 been disabled. 
  
 .EXAMPLE 
 Disable-RollbackPolicy -Computername DC1,server1,server2 -Passthru 
  
 Rollback                                Computer                                
 --------                                --------                                
 Disabled                                DC1     
 Disabled                                server1   
 Disabled                                server2   
  
 Description 
 ----------- 
 This disables the rollback policy on DC1,server1 and server2 and returns the object showing that the policy has  
 been disabled. 
 #> 
 [cmdletbinding( 
     SupportsShouldProcess = 'True' 
     )] 
 Param ( 
     [parameter()] 
     [string[]]$Computername, 
     [parameter()] 
     [Switch]$Passthru 
     ) 
 Process { 
     ForEach ($computer in $computername) { 
         Try { 
             Write-Verbose "Making registry connection to remote computer"      
             $registry = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey("LocalMachine",$computer) 
             If ($PScmdlet.ShouldProcess("$computer","Disable Rollback")) { 
                 Write-Verbose "Making registry connection to 'Software' key"  
                 If ($registry.GetSubkeyNames() -notcontains "Software") { 
                     $registry.CreateSubKey("Software") 
                     $subkey = $registry.OpenSubKey("Software",$True) 
                     } 
                 Else { 
                     $subkey = $registry.OpenSubKey("Software",$True) 
                     } 
                 Write-Verbose "Making registry connection to 'Policies' key" 
                 If ($subkey.GetSubKeyNames() -notcontains "Policies") { 
                     $subkey.CreateSubKey("Policies") 
                     $subkey = $subkey.OpenSubKey("Policies",$True) 
                     } 
                 Else { 
                     $subkey = $subkey.OpenSubKey("Policies",$True) 
                     } 
                 Write-Verbose "Making registry connection to 'Microsoft' key" 
                 If ($subkey.GetSubKeyNames() -notcontains "Microsoft") { 
                     $subkey.CreateSubKey("Microsoft") 
                     $subkey = $subkey.OpenSubKey("Microsoft",$True) 
                     } 
                 Else { 
                     $subkey = $subkey.OpenSubKey("Microsoft",$True) 
                     } 
                 Write-Verbose "Making registry connection to 'Windows' key" 
                 If ($subkey.GetSubKeyNames() -notcontains "Windows") { 
                     $subkey.CreateSubKey("Windows") 
                     $subkey = $subkey.OpenSubKey("Windows",$True) 
                     } 
                 Else { 
                     $subkey = $subkey.OpenSubKey("Windows",$True) 
                     } 
                 Write-Verbose "Making registry connection to 'Installer' key" 
                 If ($subkey.GetSubKeyNames() -notcontains "Installer") { 
                     $subkey.CreateSubKey("Installer") 
                     $subkey = $subkey.OpenSubKey("Installer",$True) 
                     } 
                 Else { 
                     $subkey = $subkey.OpenSubKey("Installer",$True) 
                     } 
                 Write-Verbose "Checking to see if 'DisableRollback' value name exists" 
                 If ($subkey.GetValueNames() -notcontains "DisableRollback") { 
                     Write-Verbose "Creating DisableRollback value name and setting to 1 (Disable)" 
                     $subkey.SetValue("DisableRollback","1","DWord")  
                     }                                  
                 Else { 
                     Write-Verbose "Disabling Rollback" 
                     $subkey.SetValue("DisableRollback","1","DWord")  
                     } 
                 If ($PSBoundParameters['Passthru']) { 
                     Write-Verbose "Retrieving value of 'DisableRollback'" 
                     $value = $subkey.GetValue('DisableRollback') 
                     Switch ($value) { 
                         0 {$hash = @{Computer = $computer;Rollback = "Enabled"}} 
                         1 {$hash = @{Computer = $computer;Rollback = "Disabled"}} 
                         Default {$hash = @{Computer = $computer;Rollback = "Unknown"}} 
                         }                      
                     }                     
                 } 
             } 
         Catch { 
             Write-Warning "$($computer): $($Error[0])" 
             } 
         Finally { 
             If ($PSBoundParameters['Passthru']) {         
                 $object = New-Object PSObject -Property $hash 
                 $object.PSTypeNames.Insert(0,'RollbackStatus') 
                 Write-Output $object    
                 } 
             }             
         } 
     } 
 } 
  
 Function Enable-RollbackPolicy { 
 .SYNOPSIS   
      Enables the rollback policy on a local or remote computer 
      
 .DESCRIPTION   
     Enables the rollback policy on a local or remote computer 
      
 .PARAMETER Computername 
     Name of computer or computers to enable the rollback policy 
      
 .PARAMETER Passthru 
     Displays the object after enabling the rollback policy on a computer 
             
 .NOTES   
     Name: Enable-RollbackPolicy 
     Author: Boe Prox 
     DateCreated: 06/24/2011 
     Links:  
  
 .EXAMPLE 
 Enable-RollbackPolicy -Computername DC1 -Passthru 
  
 Rollback                                Computer                                
 --------                                --------                                
 Enable                                DC1     
  
 Description 
 ----------- 
 This enables the rollback policy on DC1 and returns the object showing that the policy has  
 been enabled. 
  
 .EXAMPLE 
 Enable-RollbackPolicy -Computername DC1,server1,server2 -Passthru 
  
 Rollback                                Computer                                
 --------                                --------                                
 Enable                                DC1     
 Enable                                server1     
 Enable                                server2     
  
 Description 
 ----------- 
 This enables the rollback policy on DC1,server1 and server2 and returns the object showing that the policy has  
 been enabled. 
 #> 
 [cmdletbinding( 
     SupportsShouldProcess = 'True' 
     )] 
 Param ( 
     [parameter()] 
     [string[]]$Computername, 
     [parameter()] 
     [Switch]$Passthru 
     ) 
 Process { 
     ForEach ($computer in $computername) { 
         Try { 
             Write-Verbose "Making registry connection to remote computer"      
             $registry = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey("LocalMachine",$computer) 
             If ($PScmdlet.ShouldProcess("$computer","Enable Rollback")) { 
                 Write-Verbose "Making registry connection to 'Software' key"  
                 If ($registry.GetSubkeyNames() -notcontains "Software") { 
                     $registry.CreateSubKey("Software") 
                     $subkey = $registry.OpenSubKey("Software",$True) 
                     } 
                 Else { 
                     $subkey = $registry.OpenSubKey("Software",$True) 
                     } 
                 Write-Verbose "Making registry connection to 'Policies' key" 
                 If ($subkey.GetSubKeyNames() -notcontains "Policies") { 
                     $subkey.CreateSubKey("Policies") 
                     $subkey = $subkey.OpenSubKey("Policies",$True) 
                     } 
                 Else { 
                     $subkey = $subkey.OpenSubKey("Policies",$True) 
                     } 
                 Write-Verbose "Making registry connection to 'Microsoft' key" 
                 If ($subkey.GetSubKeyNames() -notcontains "Microsoft") { 
                     $subkey.CreateSubKey("Microsoft") 
                     $subkey = $subkey.OpenSubKey("Microsoft",$True) 
                     } 
                 Else { 
                     $subkey = $subkey.OpenSubKey("Microsoft",$True) 
                     } 
                 Write-Verbose "Making registry connection to 'Windows' key" 
                 If ($subkey.GetSubKeyNames() -notcontains "Windows") { 
                     $subkey.CreateSubKey("Windows") 
                     $subkey = $subkey.OpenSubKey("Windows",$True) 
                     } 
                 Else { 
                     $subkey = $subkey.OpenSubKey("Windows",$True) 
                     } 
                 Write-Verbose "Making registry connection to 'Installer' key" 
                 If ($subkey.GetSubKeyNames() -notcontains "Installer") { 
                     $subkey.CreateSubKey("Installer") 
                     $subkey = $subkey.OpenSubKey("Installer",$True) 
                     } 
                 Else { 
                     $subkey = $subkey.OpenSubKey("Installer",$True) 
                     } 
                 Write-Verbose "Checking to see if 'DisableRollback' value name exists" 
                 If ($subkey.GetValueNames() -notcontains "DisableRollback") { 
                     Write-Verbose "Creating DisableRollback value name and setting to 0 (Enable)" 
                     $subkey.SetValue("DisableRollback","0","DWord")  
                     }                                  
                 Else { 
                     Write-Verbose "Enabling Rollback" 
                     $subkey.SetValue("DisableRollback","0","DWord")  
                     } 
                 If ($PSBoundParameters['Passthru']) { 
                     Write-Verbose "Retrieving value of 'DisableRollback'" 
                     $value = $subkey.GetValue('DisableRollback') 
                     Switch ($value) { 
                         0 {$hash = @{Computer = $computer;Rollback = "Enabled"}} 
                         1 {$hash = @{Computer = $computer;Rollback = "Disabled"}} 
                         Default {$hash = @{Computer = $computer;Rollback = "Unknown"}} 
                         }                      
                     } 
                 } 
             } 
         Catch { 
             Write-Warning "$($computer): $($Error[0])" 
             } 
         Finally { 
             If ($PSBoundParameters['Passthru']) { 
                 $object = New-Object PSObject -Property $hash 
                 $object.PSTypeNames.Insert(0,'RollbackStatus') 
                 Write-Output $object    
                 } 
             } 
         } 
     } 
 } 
  
 Export-ModuleMember -Function Get-RollbackPolicy,Enable-RollbackPolicy,Disable-RollbackPolicy
`

