---
Author: dexterposh
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 5174
Published Date: 2014-05-20t22
Archived Date: 2014-05-24t12
---

# get-sccmusercomputer - 

## Description

in the configmgr environment while performing application deployments. sometimes users don’t mention their machine names.

## Comments



## Usage



## TODO



## function

`get-sccmusercomputer`

## Code

`#
 #
 function Get-SCCMUserComputer
 {
 <#
 	.SYNOPSIS
 		Gets the NetbiosName for a User (name or SamAccountName)
 
 	.DESCRIPTION
 		Retrieves the NetbiosName property on the SMS_R_System class based on the LastlogonUsername as filter.
         Can use the SamAccountNames or a Name to get the machine name, uses ADSI To find the Users when Name is specified.
 
 	
     .EXAMPLE
         Get-SCCMUserComputer -identity dexter,Administrator
 
         SamAccountName                          ComputerName                           
         --------------                          ------------                           
         dexter                                  Dexter-PC                         
         Administrator                           DexterDC
         
         Multiple SamAccountNames can be passed as an input.
         By default assumes the localmachine has SMS Provider installed.
 
 	.EXAMPLE
         Get-SCCMUserComputer -Name "Dexter POSH" -SCCMServer DexSCCM
 
         Name                       SamAccountName             ComputerName             
         ----                       --------------             ------------             
         {Dexter POSH}              {dexter}                   Dexter-PC
         
         When using the Name parameter we can specify the name like below and the space is replaced with a wild card.
         ADSI filter used to search for below is 'Dexter*POSH'. Out-Gridview will prompt you to select a User you want
         the machine name for. Specify the remote SCCM Server with SMS namespace installed.                 
 
     .EXAMPLE
         This one works the same way but the filter used is "*Dexter*" and this makes the search very slow.
         PS C:\> Get-SCCMUserComputer -Name "Dexter" -SCCMServer DexSCCM
 
         Name                       SamAccountName             ComputerName             
         ----                       --------------             ------------             
         {Dexter POSH}              {dexter}                   Dexter-PC
                   
     .EXAMPLE
                 
          'dexter','Administrator' | Get-SCCMUserComputer -SCCMServer DexSCCM
 
         SamAccountName                          ComputerName                           
         --------------                          ------------                           
         dexter                                  Dexter-PC                         
         Administrator                           DexterDC
 
         One can pipe the SamAccountNames to the Function as well 
     .INPUTS
         System.String[]
 
 	.OUTPUTS
 		PSObject[]
 
 	.NOTES
 	    Written by - DexterPOSH
         Blog Url - http://dexterposh.blogspot.com
 #>
 [CmdletBinding(DefaultParameterSetName="Identity")]
 [OutputType([PSObject])]
     Param
     (
         [Parameter(Mandatory=$true,
                    ValueFromPipeline,
                    ValueFromPipelineByPropertyName,
                    ParameterSetName="Identity")]
         [string[]]$identity,
 
        [Parameter(Mandatory=$true,
                    ValueFromPipelineByPropertyName=$true,
                    ParameterSetName="Name")]
         [string]$Name,
 
         [Parameter(Mandatory=$false)]
         [Alias("SMSProvider")]
         [String]$SCCMServer=$env:COMPUTERNAME
 
     )
 
     Begin
     {
         $CIMSessionParams = @{
                     ComputerName = $SCCMServer
                     ErrorAction = 'Stop'
                     
                 }
         try
         {
             If ((Test-WSMan -ComputerName $SCCMServer -ErrorAction SilentlyContinue).ProductVersion -match 'Stack: 3.0')
             {
                 Write-Verbose -Message "[BEGIN] WSMAN is responsive"
                 $CimSession = New-CimSession @CIMSessionParams
                 $CimProtocol = $CimSession.protocol
                 Write-Verbose -Message "[BEGIN] [$CimProtocol] CIM SESSION - Opened"
             } 
 
             else 
             {
                 Write-Verbose -Message "[PROCESS] Attempting to connect with protocol: DCOM"
                 $CIMSessionParams.SessionOption = New-CimSessionOption -Protocol Dcom
                 $CimSession = New-CimSession @CIMSessionParams
                 $CimProtocol = $CimSession.protocol
 
                 Write-Verbose -Message "[BEGIN] [$CimProtocol] CIM SESSION - Opened"
             }
        
 
 
            
             $sccmProvider = Get-CimInstance -query "select * from SMS_ProviderLocation where ProviderForLocalSite = true" -Namespace "root\sms" -CimSession $CimSession -ErrorAction Stop
             $Splits = $sccmProvider.NamespacePath -split "\\", 4
             Write-Verbose "[BEGIN] Provider is located on $($sccmProvider.Machine) in namespace $($splits[3])"
  
             $hash= @{"CimSession"=$CimSession;"NameSpace"=$Splits[3];"ErrorAction"="Stop"}
                                   
             
         }
         catch
         {
             Write-Warning "[BEGIN] $SCCMServer needs to have SMS Namespace Installed"
             throw $Error[0].Exception
         }
     }
     Process
     {
         Switch -exact ($PSCmdlet.ParameterSetName)
         {
             "Identity"
             {
                 foreach ($id in $identity)
                 {
                     $query = "Select NetbiosName from {0} where LastlogonUserName='{1}'" -f "SMS_R_System",$id
                     Get-CimInstance -Query $query @hash -PipelineVariable UserComputer | 
                         foreach -Process {
                                             [pscustomobject]@{
                                                                                                                             
                                                                 SamAccountName = $id
                                                                 ComputerName = $userComputer.netbiosname
                                                                 }}
                                             
                 }
             }
             
             "Name"
             {
                 $adsisearcher = New-Object -TypeName System.DirectoryServices.DirectorySearcher
                 if ($Name -notmatch '\s+')
                 {
                 }
                 $adsisearcher.Filter ='(&(objectCategory=person)(objectClass=user)(name={0}))' -f $($($name -replace '\s+',' ')  -replace ' ','*')
 
                 $users = $adsisearcher.FindAll() 
                 if ($users.count -ne 0)
                 {
                     if ($users.Count -ne 1)
                     {
                      $users = $users | select -ExpandProperty properties | 
                                 foreach { [pscustomobject]@{Name=$_.name;SamAccountName=$_.samaccountname;Email=$_.mail;Title=$_.title;Location=$_.l} } |
                                     Out-GridView -OutputMode Single -Title "Select the User"
                     }
                     
                 
                 $query = "Select NetbiosName from {0} where LastlogonUserName='{1}'" -f "SMS_R_System",$($users.samaccountname)
                 Get-CimInstance -Query $query @hash -PipelineVariable UserComputer | 
                     foreach -Process {
                                         [pscustomobject]@{
                                                             Name=$users.name
                                                             SamAccountName = $users.samaccountname
                                                             ComputerName = $userComputer.netbiosname
                                                             }}
                                     
                 }
                 else
                 {
                     Write-Warning -Message "No Users could be found"
                 }
             }
         }
     }
     End
     {
         Write-Verbose -Message "[END] Ending the Function"
     }
 }
`

