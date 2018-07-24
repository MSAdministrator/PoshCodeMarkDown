---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.4
Encoding: ascii
License: cc0
PoshCode ID: 464
Published Date: 
Archived Date: 2008-11-20t12
---

# get/set signature (ctp2) - 

## Description

wrappers for the get-authenticodesignature and set-authenticodesignature which properly parse paths and don’t kill your pipeline and script when you hit a folder by accident…

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ####################################################################################################
 ##
 ##
 ##
 ##
 ####################################################################################################
 ##
 ####################################################################################################
 ####################################################################################################
 
 CMDLET Get-UserCertificate -snapin Huddled.Authenticode {
    trap {
       Write-Host "The authenticode script module requires a configuration file to function fully!"
       Write-Host
       Write-Host "You must put the path to your default signing certificate in the configuration"`
                  "file before you can use the module's Set-Authenticode cmdlet or to the 'mine'"`
                  "feature of the Select-Signed or Test-Signature. To set it up, you can do this:"
       Write-Host 
       Write-Host "MkDir $(Join-Path $PsScriptRoot $(Get-Culture)) |"
       Write-Host "   Join-Path -Path {`$_} 'Authenticode.psd1'    |"
       Write-Host "   New-Item  -Path {`$_} -Type File -Value '`"ThePathToYourCertificate`"'"
       Write-Host
       Write-Host "If you load your certificate into your 'CurrentUser\My' store, or put the .pfx file"`
                  "into the folder with the Authenthenticode module script, you can just specify it's"`
                  "thumprint or filename, respectively. Otherwise, it should be a full path."
       return
    }   
    Import-LocalizedData -bindingVariable CertificatePath -EA "STOP"
    
    $ResolvedPath = $Null
    $ResolvedPath = Resolve-Path $CertificatePath -ErrorAction "SilentlyContinue"
    if(!$ResolvedPath) { 
       $ResolvedPath = Resolve-Path (Join-Path $PsScriptRoot $CertificatePath -ErrorAction "SilentlyContinue") -ErrorAction "SilentlyContinue" 
    }
    if(!$ResolvedPath) { 
       $ResolvedPath = Resolve-Path (Join-Path "Cert:\CurrentUser\My" $CertificatePath -ErrorAction "SilentlyContinue") -ErrorAction "SilentlyContinue" 
    }
 
    $Certificate = get-item $ResolvedPath -ErrorAction "SilentlyContinue"
    if($Certificate -is [System.IO.FileInfo]) {
       $Certificate = Get-PfxCertificate $Certificate -ErrorAction "SilentlyContinue"
    }
    return $Certificate
 }
 
 $DefaultCertificate = Get-UserCertificate
 $CertificateThumbprint = $DefaultCertificate.Thumbprint
 
 
 CMDLET Test-Signature -snapin Huddled.Authenticode {
 PARAM (
    [Parameter(Position=0, Mandatory=$true, ValueFromPipeline=$true)]
    $Signature
 ,
    [Alias("Valid")]
    [Switch]$ForceValid
 )
 
 return ( $Signature.Status -eq "Valid" -or 
       ( !$ForceValid -and 
          ($Signature.Status -eq "UnknownError") -and 
          ($_.SignerCertificate.Thumbprint -eq $CertificateThumbprint) 
       ) )
 }
 
 CMDLET Set-AuthenticodeSignature -snapin Huddled.Authenticode {
 PARAM ( 
    [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [ValidateScript({ 
       if((resolve-path $_).Provider.Name -ne "FileSystem") {
          throw "Specified Path is not in the FileSystem: '$_'" 
       }
       if(!(Test-Path -PathType Leaf $_)) { 
          throw "Specified Path is not a File: '$_'" 
       }
       return $true
    })]
    [string]
    $Path
    [Parameter(Position=2, Mandatory=$false)]
    $Certificate = $DefaultCertificate
 )
 
 PROCESS {
    if( ".psm1" -eq [IO.Path]::GetExtension($Path) ) {
       $ps1Path = "$Path.ps1"
       Rename-Item $Path (Split-Path $ps1Path -Leaf)
       $sig = Microsoft.PowerShell.Security\Set-AuthenticodeSignature -Certificate $Certificate -FilePath $ps1Path | Select *
       Rename-Item $ps1Path (Split-Path $Path -Leaf) 
       $sig.PSObject.TypeNames.Insert( 0, "System.Management.Automation.Signature" )
       $sig.Path = $Path
       $sig
    } else {
       Microsoft.PowerShell.Security\Set-AuthenticodeSignature -Certificate $Certificate -FilePath $Path  
    }
 }
 }
 
 CMDLET Get-AuthenticodeSignature -snapin Huddled.Authenticode {
 PARAM ( 
    [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [ValidateScript({ 
       if((resolve-path $_).Provider.Name -ne "FileSystem") {
          throw "Specified Path is not in the FileSystem: '$_'" 
       }
       if(!(Test-Path -PathType Leaf $_)) { 
          throw "Specified Path is not a File: '$_'" 
       }
       return $true
    })]
    [string]
    $Path
 )
 
 PROCESS {
    if( ".psm1" -eq [IO.Path]::GetExtension($Path) ) {
       $ps1Path = "$Path.ps1"
       Rename-Item $Path (Split-Path $ps1Path -Leaf)
       $sig = Microsoft.PowerShell.Security\Get-AuthenticodeSignature -FilePath $ps1Path | select *
       Rename-Item $ps1Path (Split-Path $Path -Leaf) 
       $sig.PSObject.TypeNames.Insert( 0, "System.Management.Automation.Signature" )
       $sig.Path = $Path
       $sig
    } else {
       Microsoft.PowerShell.Security\Get-AuthenticodeSignature -FilePath $Path
    }
 
 }
 }
 
 CMDLET Select-Signed -snapin Huddled.Authenticode {
 PARAM ( 
    [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [ValidateScript({ 
       if((resolve-path $_).Provider.Name -ne "FileSystem") {
          throw "Specified Path is not in the FileSystem: '$_'" 
       }
       return $true
    })]
    [string]
    $Path
 ,
    [Parameter()]
    [switch]$MineOnly
 ,
    [Parameter()]
    [switch]$NotMineOnly
 ,
    [Parameter()]
    [switch]$BrokenOnly
 ,
    [Parameter()]
    [switch]$TrustedOnly
 ,
    [Parameter()]
    [switch]$ValidOnly
 ,
    [Parameter()]
    [switch]$InvalidOnly
 ,
    [Parameter()]
    [switch]$UnsignedOnly
 
 )
 
    if(!(Test-Path -PathType Leaf $Path)) { 
    } else {
 
       $sig = Get-AuthenticodeSignature $Path 
       
       if($BrokenOnly   -and $sig.Status -ne "HashMismatch") 
       { 
          Write-Debug "$($sig.Status) - Not Broken: $Path"
          return 
       }
       
       if($TrustedOnly  -and $sig.Status -ne "Valid") 
       { 
          Write-Debug "$($sig.Status) - Not Trusted: $Path"
          return 
       }
       
       if($ValidOnly    -and (($sig.Status -ne "HashMismatch") -or !$sig.SignerCertificate) ) 
       { 
          Write-Debug "$($sig.Status) - Not Valid: $Path"
          return 
       }
       
       if($InvalidOnly  -and ($sig.Status -eq "Valid")) 
       { 
          Write-Debug "$($sig.Status) - Valid: $Path"
          return 
       }
       
       if($UnsignedOnly -and $sig.SignerCertificate ) 
       { 
          Write-Debug "$($sig.Status) - Signed: $Path"
          return 
       }
       
       if($MineOnly     -and (!($sig.SignerCertificate) -or ($sig.SignerCertificate.Thumbprint -ne $CertificateThumbprint)))
       {
          Write-Debug "Originally signed by someone else, thumbprint: $($sig.SignerCertificate.Thumbprint)"
          Write-Debug "Does not match your default certificate print: $CertificateThumbprint"
          Write-Debug "     $Path"
          return 
       }
 
       if($NotMineOnly  -and (!($sig.SignerCertificate) -or ($sig.SignerCertificate.Thumbprint -eq $CertificateThumbprint)))
       {
          if($sig.SignerCertificate) {
             Write-Debug "Originally signed by you, thumbprint: $($sig.SignerCertificate.Thumbprint)"
             Write-Debug "Matches your default certificate print: $CertificateThumbprint"
             Write-Debug "     $Path"
          }
          return 
       }
       
       if(!$BrokenOnly  -and !$TrustedOnly -and !$ValidOnly -and !$InvalidOnly -and !$UnsignedOnly -and !($sig.SignerCertificate) ) 
       { 
          Write-Debug "$($sig.Status) - Not Signed: $Path"
          return 
       }
       
       get-childItem $sig.Path
    }
 }
 
 Export-ModuleMember Set-AuthenticodeSignature,Get-AuthenticodeSignature,Test-Signature,Select-Signed,Get-UserCertificate
 
`

