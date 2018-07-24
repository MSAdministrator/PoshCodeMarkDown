---
Author: dexterposh
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 5048
Published Date: 2014-04-03t05
Archived Date: 2014-04-06t17
---

# remove-sccmdpcontent - 

## Description

the function remove-sccmdpcontent will remove a list of packageids from a distribution point.

## Comments



## Usage



## TODO



## function

`remove-sccmdpcontent`

## Code

`#
 #
 function Remove-SCCMDPContent
 {
 <#
 .Synopsis
     Function to Remove Packages from the DP
 .DESCRIPTION
     THis Function will connect to the SCCM Server SMS namespace and then remove the Package IDs
     passed to the Function from the specified DP name.
 .EXAMPLE
     PS> Remove-SCCMDPContent -PackageID DEX123AB,DEX145CD -DPname DexDP -Computername DexSCCMServer  
 
     This will remove the Packages with Package IDs [ DEX123AB,DEX145CD] from the Distribution Point "DexDP".
 .INPUTS
     System.String[]
 .OUTPUTS
     System.Management.Automation.PSCustomObject
 .NOTES
     Author - DexterPOSH (Deepak Singh Dhami)
     BlogLink - http://dexterposh.blogspot.com/2014/02/use-powershell-to-remove-packages-from.html
 #>
 
     [CmdletBinding()]
     [OutputType([PSObject])]
     Param
     (
         [Parameter(Mandatory,
                    ValueFromPipelineByPropertyName,
                    Position=0)]
         [string[]]$PackageID,
 
         [Parameter(Mandatory=$true)]
         [String]$DPName,
 
         [Parameter()]
         [Alias("SCCMServer")]
         [String]$ComputerName
     )
 
     Begin
     {
         Write-Verbose -Message "[BEGIN]"
         try
         {
            
             $sccmProvider = Get-WmiObject -query "select * from SMS_ProviderLocation where ProviderForLocalSite = true" -Namespace "root\sms" -computername $ComputerName -errorAction Stop
             $Splits = $sccmProvider.NamespacePath -split "\\", 4
             Write-Verbose "Provider is located on $($sccmProvider.Machine) in namespace $($splits[3])"
  
             $hash= @{"ComputerName"=$ComputerName;"NameSpace"=$Splits[3];"Class"="SMS_DistributionPoint";"ErrorAction"="Stop"}
             
             $hash.Add("Filter","ServerNALPath LIKE '%$DPname%'")
 
             
         }
         catch
         {
             Write-Warning "Something went wrong while getting the SMS ProviderLocation"
             throw $Error[0].Exception
         }
     }
     Process
     {
         
             
             Write-Verbose -Message "[PROCESS] Working to remove packages from DP --> $DPName  "
             
             $PackagesINDP = Get-WmiObject @hash
             
             $RemovePackages = $PackagesINDP | where {$PackageID -contains $_.packageid   }
             
             
             $Removepackages | ForEach-Object -Process { 
                 try 
                 {
                     Remove-WmiObject  -InputObject $_  -ErrorAction Stop -ErrorVariable WMIRemoveError 
                     Write-Verbose -Message "Removed $($_.PackageID) from $DPname"
                     [pscustomobjet]@{"DP"=$DPname;"PackageId"=$($_.PackageID);"Action"="Removed"}
                                                                  
                 }
                 catch
                 {
                     Write-Warning "[PROCESS] Something went wrong while removing the Package  from $DPname"
                     $WMIRemoveError.Exception
                 }
             
     End
     {
         Write-Verbose "[END] Ending the Function"
     }
 }
`

