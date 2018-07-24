---
Author: zachary loeber
Publisher: 
Copyright: 
Email: 
Version: 1.0.0
Encoding: ascii
License: cc0
PoshCode ID: 4368
Published Date: 
Archived Date: 2013-08-13t09
---

#  - 

## Description

a multithreaded remote registry gathering function. includes the ability to return both a specific subkey value or an entire key of subkey values in a custom psobject.

## Comments



## Usage



## TODO



## function

`get-remoteregistryinformation`

## Code

`#
 #
 function Get-RemoteRegistryInformation
 {
 <#
 .SYNOPSIS
    Retrieves registry subkey information.
 .DESCRIPTION
    Retrieves registry subkey information. If an explicit subkey is passed then that subkey value
    is returned. Otherwise, all subkeys and their values are returned as a custom psobject.
 .PARAMETER ComputerName
    Specifies the target computer for data query.
 .PARAMETER Hive
    Registry hive to retrieve from. By default this is 2147483650 (HKLM). Valid hives include:
       HKEY_CLASSES_ROOT = 2147483648
       HKEY_CURRENT_USER = 2147483649
       HKEY_LOCAL_MACHINE = 2147483650
       HKEY_USERS = 2147483651
       HKEY_CURRENT_CONFIG = 2147483653
       HKEY_DYN_DATA = 2147483654
 .PARAMETER Key
    Registry key to inspect (ie. SYSTEM\CurrentControlSet\Services\W32Time\Parameters)
 .PARAMETER SubKey
    Registry subkey to return data from. If this is not passed then all subkeys within the key being
    inspected are returned as a custom psobject.
 .PARAMETER ThrottleLimit
    Specifies the maximum number of systems to inventory simultaneously 
 .PARAMETER Timeout
    Specifies the maximum time in second command can run in background before terminating this thread.
 .PARAMETER ShowProgress
    Show progress bar information
 .EXAMPLE
    PS > Get-RemoteRegistryInformation -Key "SYSTEM\CurrentControlSet\Services\W32Time\Parameters" -Subkey 'Type'
 
    NT5DS
    
    Description
    -----------
    Return the value of the 'Type' subkey within SYSTEM\CurrentControlSet\Services\W32Time\Parameters of
    HKLM.
 .EXAMPLE
    PS > $b = Get-RemoteRegistryInformation -Key "SYSTEM\CurrentControlSet\Services\W32Time\Parameters"
    PS > $b | Select SubKey,SubKeyValue
    
     SubKey                                                         SubKeyValue                                                  
     ------                                                         -----------                                                  
     ServiceDll                                                     C:\Windows\system32\w32time.dll
     ServiceMain                                                    SvchostEntry_W32Time
     ServiceDllUnloadOnStop                                                             
     Type                                                           NT5DS               
     NtpServer                                                             
 
    Description
    -----------
    Return subkeys and their values within SYSTEM\CurrentControlSet\Services\W32Time\Parameters of
    HKLM.
 
 .NOTES
    Author: Zachary Loeber
    Site: http://www.the-little-things.net/
    Requires: Powershell 2.0
 
    Version History
    1.0.0 - 08/06/2013
     - Initial release
 #>
     [CmdletBinding()]
     Param
     (
         [Parameter(HelpMessage="Computer or computers to gather information from",
                    ValueFromPipeline=$true,
                    ValueFromPipelineByPropertyName=$true,
                    Position=0)]
         [ValidateNotNullOrEmpty()]
         [Alias('DNSHostName','PSComputerName')]
         [string[]]
         $ComputerName=$env:computername,
         
         [Parameter( HelpMessage="Registry Hive (Default is HKLM)." )]
     	[UInt32]
         $Hive = 2147483650,
         
     	[Parameter( Mandatory=$true,
                     HelpMessage="Registry Key to inspect." )]
     	[String]
         $Key,
         
     	[Parameter( HelpMessage="Registry Subkey to retrieve. Do not pass this and all subkeys within the key will be returned instead." )]
     	[String]
         $SubKey = '',
        
         [Parameter(HelpMessage="Maximum number of concurrent threads.")]
         [ValidateRange(1,65535)]
         [int32]
         $ThrottleLimit = 32,
  
         [Parameter(HelpMessage="Timeout before a thread stops trying to gather the information.")]
         [ValidateRange(1,65535)]
         [int32]
         $Timeout = 120,
  
         [Parameter(HelpMessage="Display progress of function.")]
         [switch]
         $ShowProgress,
         
         [Parameter(HelpMessage="Set this if you want the function to prompt for alternate credentials.")]
         [switch]
         $PromptForCredential,
         
         [Parameter(HelpMessage="Set this if you want to provide your own alternate credentials.")]
         [System.Management.Automation.Credential()]
         $Credential = [System.Management.Automation.PSCredential]::Empty
     )
 
     Begin
     {
         Write-Verbose -Message 'Creating local hostname list'
         $IPAddresses = [net.dns]::GetHostAddresses($env:COMPUTERNAME) | Select-Object -ExpandProperty IpAddressToString
         $HostNames = $IPAddresses | ForEach-Object {
             try {
                 [net.dns]::GetHostByAddress($_)
             } catch {
             }
         } | Select-Object -ExpandProperty HostName -Unique
         $LocalHost = @('', '.', 'localhost', $env:COMPUTERNAME, '::1', '127.0.0.1') + $IPAddresses + $HostNames
  
         Write-Verbose -Message 'Creating initial variables'
         $runspacetimers       = [HashTable]::Synchronized(@{})
         $runspaces            = New-Object -TypeName System.Collections.ArrayList
         $bgRunspaceCounter    = 0
         
         if ($PromptForCredential)
         {
             $Credential = Get-Credential
         }
         
         Write-Verbose -Message 'Creating Initial Session State'
         $iss = [System.Management.Automation.Runspaces.InitialSessionState]::CreateDefault()
         foreach ($ExternalVariable in ('runspacetimers', 'Credential', 'LocalHost'))
         {
             Write-Verbose -Message "Adding variable $ExternalVariable to initial session state"
             $iss.Variables.Add((New-Object -TypeName System.Management.Automation.Runspaces.SessionStateVariableEntry -ArgumentList $ExternalVariable, (Get-Variable -Name $ExternalVariable -ValueOnly), ''))
         }
         
         Write-Verbose -Message 'Creating runspace pool'
         $rp = [System.Management.Automation.Runspaces.RunspaceFactory]::CreateRunspacePool(1, $ThrottleLimit, $iss, $Host)
         $rp.Open()
  
         Write-Verbose -Message 'Defining background runspaces scriptblock'
         $ScriptBlock = {
             [CmdletBinding()]
             Param
             (
                 [Parameter(Position=0)]
                 [string]
                 $ComputerName,
 
                 [Parameter()]
             	[UInt32]
                 $Hive = 2147483650,
                 
             	[Parameter()]
             	[String]
                 $Key,
                 
             	[Parameter()]
             	[String]
                 $SubKey = '',
 
                 [Parameter()]
                 [int]
                 $bgRunspaceID
             )
             $runspacetimers.$bgRunspaceID = Get-Date
             
             try
             {
                 Write-Verbose -Message ('Runspace {0}: Start' -f $ComputerName)
                 $WMIHast = @{
                     ComputerName = $ComputerName
                     ErrorAction = 'Stop'
                 }
                 if (($LocalHost -notcontains $ComputerName) -and ($Credential -ne $null))
                 {
                     $WMIHast.Credential = $Credential
                 }
 
                 $PSDateTime = Get-Date
                 
                 Write-Verbose -Message ('Runspace {0}: Registry information' -f $ComputerName)
 
                 $wmi_data = Get-WmiObject @WMIHast -Class StdRegProv -Namespace 'root\default' -List:$true
                 
                 if ($SubKey -ne '')
                 {
                     $subkeys = $wmi_data.GetStringValue($Hive, $Key, $Subkey)
                     $ResultObject = $subkeys.sValue
                 }
                 else
                 {
                    $ResultObject = @() #($wmi_data.EnumValues($Hive,$Key)).sNames
                    Foreach ($_subkey in (($wmi_data.EnumValues($Hive,$Key)).sNames))
                    {
                         $ResultProperty = @{
                             'PSComputerName' = $ComputerName
                             'PSDateTime' = $PSDateTime
                             'ComputerName' = $ComputerName
                             'SubKey' = $_subkey
                             'SubKeyValue' = ($wmi_data.GetStringValue($Hive, $Key, $_subkey)).sValue
                         }
                         $ResultObject += New-Object PSObject -Property $ResultProperty
                     }
                 }
 
                 Write-Output -InputObject $ResultObject
             }
             catch
             {
                 Write-Warning -Message ('{0}: {1}' -f $ComputerName, $_.Exception.Message)
             }
             Write-Verbose -Message ('Runspace {0}: End' -f $ComputerName)
         }
  
         function Get-Result
         {
             [CmdletBinding()]
             Param 
             (
                 [switch]$Wait
             )
             do
             {
                 $More = $false
                 foreach ($runspace in $runspaces)
                 {
                     $StartTime = $runspacetimers.($runspace.ID)
                     if ($runspace.Handle.isCompleted)
                     {
                         Write-Verbose -Message ('Thread done for {0}' -f $runspace.IObject)
                         $runspace.PowerShell.EndInvoke($runspace.Handle)
                         $runspace.PowerShell.Dispose()
                         $runspace.PowerShell = $null
                         $runspace.Handle = $null
                     }
                     elseif ($runspace.Handle -ne $null)
                     {
                         $More = $true
                     }
                     if ($Timeout -and $StartTime)
                     {
                         if ((New-TimeSpan -Start $StartTime).TotalSeconds -ge $Timeout -and $runspace.PowerShell)
                         {
                             Write-Warning -Message ('Timeout {0}' -f $runspace.IObject)
                             $runspace.PowerShell.Dispose()
                             $runspace.PowerShell = $null
                             $runspace.Handle = $null
                         }
                     }
                 }
                 if ($More -and $PSBoundParameters['Wait'])
                 {
                     Start-Sleep -Milliseconds 100
                 }
                 foreach ($threat in $runspaces.Clone())
                 {
                     if ( -not $threat.handle)
                     {
                         Write-Verbose -Message ('Removing {0} from runspaces' -f $threat.IObject)
                         $runspaces.Remove($threat)
                     }
                 }
                 if ($ShowProgress)
                 {
                     $ProgressSplatting = @{
                         Activity = 'Getting asset info'
                         Status = '{0} of {1} total threads done' -f ($bgRunspaceCounter - $runspaces.Count), $bgRunspaceCounter
                         PercentComplete = ($bgRunspaceCounter - $runspaces.Count) / $bgRunspaceCounter * 100
                     }
                     Write-Progress @ProgressSplatting
                 }
             }
             while ($More -and $PSBoundParameters['Wait'])
         }
     }
     Process
     {
         foreach ($Computer in $ComputerName)
         {
             $bgRunspaceCounter++
             #$psCMD = [System.Management.Automation.PowerShell]::Create().AddScript($ScriptBlock).AddParameter('bgRunspaceID',$bgRunspaceCounter).AddParameter('ComputerName',$Computer).AddParameter('IncludeMemoryInfo',$IncludeMemoryInfo).AddParameter('IncludeDiskInfo',$IncludeDiskInfo).AddParameter('IncludeNetworkInfo',$IncludeNetworkInfo).AddParameter('Verbose',$Verbose)
             $psCMD = [System.Management.Automation.PowerShell]::Create().AddScript($ScriptBlock)
             $null = $psCMD.AddParameter('bgRunspaceID',$bgRunspaceCounter)
             $null = $psCMD.AddParameter('ComputerName',$Computer)
             $null = $psCMD.AddParameter('Hive',$Hive)
             $null = $psCMD.AddParameter('Key',$Key)
             $null = $psCMD.AddParameter('SubKey',$SubKey)
             $null = $psCMD.AddParameter('Verbose',$VerbosePreference)
             $psCMD.RunspacePool = $rp
  
             Write-Verbose -Message ('Starting {0}' -f $Computer)
             [void]$runspaces.Add(@{
                 Handle = $psCMD.BeginInvoke()
                 PowerShell = $psCMD
                 IObject = $Computer
                 ID = $bgRunspaceCounter
            })
            Get-Result
         }
     }
  
     End
     {
         Get-Result -Wait
         if ($ShowProgress)
         {
             Write-Progress -Activity 'Getting share session information' -Status 'Done' -Completed
         }
         Write-Verbose -Message "Closing runspace pool"
         $rp.Close()
         $rp.Dispose()
     }
 }
`

