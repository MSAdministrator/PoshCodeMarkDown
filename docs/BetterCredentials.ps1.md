---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 3.0
Encoding: ascii
License: cc0
PoshCode ID: 4373
Published Date: 2013-08-07t21
Archived Date: 2013-08-13t09
---

# bettercredentials - 

## Description

this module replaces get-credential with a better version that supports customizing the prompt dialog.

## Comments



## Usage



## TODO



## function

`get-credential`

## Code

`#
 #
 function Get-Credential { 
    [CmdletBinding(DefaultParameterSetName="Prompted")]
    param(
       [Parameter(ParameterSetName="Prompted",Position=1,Mandatory=$false)]
       [Parameter(ParameterSetName="Promptless",Position=1,Mandatory=$true)]
       [Parameter(ParameterSetName="StoreCreds",Position=1,Mandatory=$true)]
       [Parameter(ParameterSetName="Flush",Position=1,Mandatory=$true)]
       [Alias("Credential")]
       [PSObject]$UserName=$null,
 
       #
       [Parameter(ParameterSetName="Prompted",Position=2,Mandatory=$false)]
       [string]$Title=$null,
 
       [Parameter(ParameterSetName="Prompted",Position=3,Mandatory=$false)]
       [string]$Message=$null,
 
       [Parameter(ParameterSetName="Prompted",Position=4,Mandatory=$false)]
       [Parameter(ParameterSetName="Promptless",Position=4,Mandatory=$false)]
       [string]$Domain=$null,
 
       [Parameter(ParameterSetName="Prompted",Mandatory=$false)]
       [switch]$GenericCredentials,
 
       [Parameter(ParameterSetName="Prompted",Mandatory=$false)]
       [switch]$Inline,
 
       [Parameter(ParameterSetName="Prompted",Mandatory=$false)]
       [Parameter(ParameterSetName="Promptless",Mandatory=$false)]
       [Parameter(ParameterSetName="StoreCreds",Mandatory=$true)]
       [switch]$Store,
 
       [Parameter(ParameterSetName="Prompted",Mandatory=$false)]
       [Parameter(ParameterSetName="Promptless",Mandatory=$false)]
       [Parameter(ParameterSetName="Flush",Mandatory=$true)]
       [switch]$Flush,
 
       [Parameter(ParameterSetName="Prompted",Mandatory=$false)]
       [Parameter(ParameterSetName="Promptless",Mandatory=$false)]
       [Parameter(ParameterSetName="StoreCreds",Mandatory=$false)]
       $CredentialFolder = $(Join-Path ${Env:APPDATA} Credentials),
 
       [Parameter(ParameterSetName="Promptless",Position=5,Mandatory=$true)]
       $Password = $(
       if($UserName -and (Test-Path "$(Join-Path $CredentialFolder $UserName).cred")) {
             if($Flush) {
                Remove-Item "$(Join-Path $CredentialFolder $UserName).cred"
             } else {
                Get-Content "$(Join-Path $CredentialFolder $UserName).cred" | ConvertTo-SecureString 
             }
       })
    )
    process {
       [Management.Automation.PSCredential]$Credential = $null
       if( $UserName -is [System.Management.Automation.PSCredential]) {
          $Credential = $UserName
       } elseif($UserName -ne $null) {
          $UserName = $UserName.ToString()
       }
       
       if($Password) {
          if($Password -isnot [System.Security.SecureString]) {
             $Password = Encode-SecureString $Password
          }
          if($Domain) {
             $Credential = New-Object System.Management.Automation.PSCredential ${Domain}\${UserName}, ${Password}
          } else {
             $Credential = New-Object System.Management.Automation.PSCredential ${UserName}, ${Password}
          }
       }
       
       if(!$Credential) {
          if($Inline) {
             if($Title)    { Write-Host $Title }
             if($Message)  { Write-Host $Message }
             if($Domain) { 
                if($UserName -and $UserName -notmatch "[@\\]") { 
                   $UserName = "${Domain}\${UserName}"
                }
             }
             if(!$UserName) {
                $UserName = Read-Host "User"
                if(($Domain -OR !$GenericCredentials) -and $UserName -notmatch "[@\\]") {
                   $UserName = "${Domain}\${UserName}"
                }
             }
             $Credential = New-Object System.Management.Automation.PSCredential $UserName,$(Read-Host "Password for user $UserName" -AsSecureString)
          } else {
             if($GenericCredentials) { $Type = "Generic" } else { $Type = "Domain" }
          
             $Credential = $Host.UI.PromptForCredential($Title, $Message, $UserName, $Domain, $Type, "Default")
          }
       }
       
       if($Store) {
          $CredentialFile = "$(Join-Path $CredentialFolder $Credential.GetNetworkCredential().UserName).cred"
          Write-Verbose "Storing Credentials in '$CredentialFile'"
          if(!(Test-Path $CredentialFolder)) {
             mkdir $CredentialFolder | out-null
          }
          $Credential.Password | ConvertFrom-SecureString | Set-Content $CredentialFile
       }
       return $Credential
    }
 }
 
 function Decode-SecureString {
    #.Synopsis
    [CmdletBinding()]
    [OutputType("System.String")]
    param(
       [Parameter(ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
       [Alias("Password")]
       [SecureString]$secure
    )
    end {
       if($secure -eq $null) { return "" }
       $BSTR = [System.Runtime.InteropServices.marshal]::SecureStringToBSTR($secure)
       Write-Output [System.Runtime.InteropServices.marshal]::PtrToStringAuto($BSTR)
       [System.Runtime.InteropServices.Marshal]::ZeroFreeBSTR($BSTR)
    }
 }
 
 function Encode-SecureString {
    #.Synopsis
    [CmdletBinding()]
    [OutputType("System.Security.SecureString")]
    param(
       [Parameter(ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
       [String]$String
    )
    end {
       [char[]]$Chars = $String.ToString().ToCharArray()
       $SecureString = New-Object System.Security.SecureString
       foreach($c in $chars) { $SecureString.AppendChar($c) }
       Write-Output $SecureString
    }
 }
 
 Export-ModuleMember -Function Get-Credential
`

