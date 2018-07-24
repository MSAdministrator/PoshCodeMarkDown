---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.7
Encoding: ascii
License: cc0
PoshCode ID: 1111
Published Date: 
Archived Date: 2009-05-19t06
---

# get/set signature (ctp3) - 

## Description

wrappers for the get-authenticodesignature and set-authenticodesignature which properly parse paths and don’t kill your pipeline and script when you hit a folder by accident…

## Comments



## Usage



## TODO



## script

`get-usercertificate`

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
 ##
 ##
 ##
 ##
 ##
 ##
 ####################################################################################################
 
 ####################################################################################################
 function Get-UserCertificate {
 <#.SYNOPSIS
  Gets the user's default signing certificate so we don't have to ask them over and over...
 .DESCRIPTION
  The Get-UserCertificate function retrieves and returns a certificate from the user. It also stores 
  the certificate so it can bereused without re-querying for the location and/or password ... 
 .RETURNVALUE
  An X509Certificate2 suitable for code-signing
 #>##################################################################################################
 [CmdletBinding()]
 PARAM ()
 PROCESS {
    trap {
       Write-Host "The authenticode script module requires configuration to function fully!"
       Write-Host
       Write-Host "You must put the path to your default signing certificate in the module metadata"`
                  "file before you can use the module's Set-Authenticode cmdlet or to the 'mine'"`
                  "feature of the Select-Signed or Test-Signature. To set it up, you can do this:"
       Write-Host 
       Write-Host "PrivateData = 'C:\Users\${Env:UserName}\Documents\WindowsPowerShell\PoshCerts\Code-Signing.pfx'"
       Write-Host
       Write-Host "If you load your certificate into your 'CurrentUser\My' store, or put the .pfx file"`
                  "into the folder with the Authenthenticode module script, you can just specify it's"`
                  "thumprint or filename, respectively. Otherwise, it should be a full path."
       Write-Error $_
       return      
    }
    if(!$ExecutionContext.SessionState.Module.PrivateData) {
       Write-Host $($ExecutionContext | out-string) -Fore Yellow
       Write-Host $($ExecutionContext.SessionState | out-string) -Fore Yellow
       Write-Host $($ExecutionContext.SessionState.Module | fl * | out-string) -Fore Yellow
       Write-Host $($ExecutionContext.SessionState.Module | gm | out-string) -Fore Yellow
       Write-Host $($ExecutionContext.SessionState.Module.PrivateData | out-string) -Fore Yellow
       $certs = @(ls Cert:\CurrentUser\My)
       if($certs.Count) {
          Write-Host "You have $($certs.Count) certs in your local cert storage which you can specify by Thumbprint:" -fore cyan
          $certs | Out-Host
       }
       $ExecutionContext.SessionState.Module.PrivateData = $(Read-Host "Please specify a user certificate")
    }
    if($ExecutionContext.SessionState.Module.PrivateData -isnot [System.Security.Cryptography.X509Certificates.X509Certificate2]) {
       $ResolvedPath = $Null
       $ResolvedPath = Resolve-Path $ExecutionContext.SessionState.Module.PrivateData -ErrorAction "SilentlyContinue"
       if(!$ResolvedPath -or !(Test-Path $ResolvedPath -ErrorAction "SilentlyContinue")) {
          $ResolvedPath = Resolve-Path (Join-Path $PsScriptRoot $ExecutionContext.SessionState.Module.PrivateData -ErrorAction "SilentlyContinue") -ErrorAction "SilentlyContinue" 
       }
       if(!$ResolvedPath -or !(Test-Path $ResolvedPath -ErrorAction "SilentlyContinue")) {
          $ResolvedPath = Resolve-Path (Join-Path "Cert:\CurrentUser\My" $ExecutionContext.SessionState.Module.PrivateData -ErrorAction "SilentlyContinue") -ErrorAction "SilentlyContinue" 
       }
 
       $Certificate = get-item $ResolvedPath -ErrorAction "SilentlyContinue"
       if($Certificate -is [System.IO.FileInfo]) {
          $Certificate = Get-PfxCertificate $Certificate -ErrorAction "SilentlyContinue"
       }
       Write-Verbose "Certificate: $($Certificate | Out-String)"
       $ExecutionContext.SessionState.Module.PrivateData = $Certificate
    }
    return $ExecutionContext.SessionState.Module.PrivateData
 }
 }
 
 function Test-Signature {
 <#
 .SYNOPSIS
  Tests a script signature to see if it is valid, or at least unaltered.
 .DESCRIPTION
  The Test-Signature function processes the output of Get-AuthenticodeSignature to determine if it 
  is Valid, OR **unaltered** and signed by the user's certificate
 .NOTES
  Test-Signature returns TRUE even if the root CA certificate can't be verified, as long as the 
  signing certificate's thumbnail matches the one specified by Get-UserCertificate.
 .EXAMPLE
    ls *.ps1 | Get-AuthenticodeSignature | Where {Test-Signature $_}
  To get the signature reports for all the scripts that we consider safely signed.
 .EXAMPLE
    ls | ? { gas $_ | test-signature }
  List all the valid signed scripts (or scripts signed by our cert)
 .INPUTTYPE
   System.Management.Automation.Signature
 .PARAMETER Signature
   Specifies the signature object to test. This should be the output of Get-AuthenticodeSignature.
 .PARAMETER ForceValid
   Switch parameter, forces the signature to be valid -- otherwise, even if the certificate chain can't be verified, we will accept the cert which matches the "user" certificate (see Get-UserCertificate).
   Aliases                      Valid
 .RETURNVALUE
    Boolean value representing whether the script's signature is valid, or YOUR certificate
 #>
 [CmdletBinding()]
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
          ($_.SignerCertificate.Thumbprint -eq $(Get-UserCertificate).Thumbprint) 
       ) )
 }
 
 
 
 ####################################################################################################
 function Set-AuthenticodeSignature {
 <#.SYNOPSIS
    Adds an Authenticode signature to a Windows PowerShell script or other file.
 .DESCRIPTION
    The Set-AuthenticodeSignature function adds an Authenticode signature to any file that supports Subject Interface Package (SIP).
  
    In a Windows PowerShell script file, the signature takes the form of a block of text that indicates the end of the instructions that are executed in the script. If there is a signature  in the file when this cmdlet runs, that signature is removed.
 .NOTES
    After the certificate has been validated, but before a signature is added to the file, the function checks the value of the $SigningApproved preference variable. If this variable is not set, or has a value other than TRUE, you are prompted to confirm the signing of the script.
  
    When specifying multiple values for a parameter, use commas to separate the values. For example, "<parameter-name> <value1>, <value2>".
 .EXAMPLE
    ls *.ps1 | Set-AuthenticodeSignature -Certificate $Certificate
    
    To sign all of the files with the specified certificate
 .EXAMPLE
    ls *.ps1,*.psm1,*.psd1 | Get-AuthenticodeSignature | Where {!(Test-Signature $_ -Valid)} | gci | Set-AuthenticodeSignature
 
    List all the script files, and get and test their signatures, and then sign all of the ones that are not valid, using the user's default certificate.
 .INPUTTYPE
    String. You can pipe a file path to Set-AuthenticodeSignature.
 .PARAMETER FilePath
    Specifies the path to a file that is being signed.
    Aliases                      Path, FullName
 .PARAMETER Certificate
    Specifies the certificate that will be used to sign the script or file. Enter a variable that stores an object representing the certificate or an expression that gets the certificate.
   
    To find a certificate, use Get-PfxCertificate or use the Get-ChildItem cmdlet in the Certificate (Cert:) drive. If the certificate is not valid or does not have code-signing authority, the command fails.
 .RETURNVALUE
    System.Management.Automation.Signature
 ###################################################################################################>
 [CmdletBinding()]
 PARAM ( 
    [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName","Path")]
    [ValidateScript({ 
       if((resolve-path $_).Provider.Name -ne "FileSystem") {
          throw "Specified Path is not in the FileSystem: '$_'" 
       }
       if(!(Test-Path -PathType Leaf $_)) { 
          throw "Specified Path is not a File: '$_'" 
       }
       return $true
    })]
    [string[]]
    $FilePath
 ,  
    [Parameter(Position=2, Mandatory=$false)]
    $Certificate = $(Get-UserCertificate)
 )
 
 PROCESS {
    Write-Verbose "Set Authenticode Signature on $FilePath with $($Certificate | Out-String)"
    Microsoft.PowerShell.Security\Set-AuthenticodeSignature -Certificate $Certificate -FilePath $(Resolve-Path $FilePath)
 }
 }
 New-Alias sign Set-AuthenticodeSignature -EA "SilentlyContinue"
 
 ####################################################################################################
 function Get-AuthenticodeSignature {
 <#.SYNOPSIS
    Gets information about the Authenticode signature in a file.
 .DESCRIPTION
    The Get-AuthenticodeSignature function gets information about the Authenticode signature in a file. If the file is not signed, the information is retrieved, but the fields are blank.
 .NOTES
    For information about Authenticode signatures in Windows PowerShell, type "get-help About_Signing".
 
    When specifying multiple values for a parameter, use commas to separate the values. For example, "-<parameter-name> <value1>, <value2>".
 .EXAMPLE
    Get-AuthenticodeSignature script.ps1
    
    To get the signature information about the script.ps1 script file.
 .EXAMPLE
    ls *.ps1,*.psm1,*.psd1 | Get-AuthenticodeSignature
    
    Get the signature information for all the script and data files
 .EXAMPLE
    ls *.ps1,*.psm1,*.psd1 | Get-AuthenticodeSignature | Where {!(Test-Signature $_ -Valid)} | gci | Set-AuthenticodeSignature
 
    This command gets information about the Authenticode signature in all of the script and module files, and tests the signatures, then signs all of the ones that are not valid.
 .INPUTTYPE
    String. You can pipe the path to a file to Get-AuthenticodeSignature.
 .PARAMETER FilePath
    The path to the file being examined. Wildcards are permitted, but they must lead to a single file. The parameter name ("-FilePath") is optional.
    Aliases                      Path, FullName
 .RETURNVALUE
    System.Management.Automation.Signature
 ###################################################################################################>
 [CmdletBinding()]
 PARAM ( 
    [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName","Path")]
    [ValidateScript({ 
       if((resolve-path $_).Provider.Name -ne "FileSystem") {
          throw "Specified Path is not in the FileSystem: '$_'" 
       }
       if(!(Test-Path -PathType Leaf $_)) { 
          throw "Specified Path is not a File: '$_'" 
       }
       return $true
    })]
    [string[]]
    $FilePath
 )
 
 PROCESS {
    Microsoft.PowerShell.Security\Get-AuthenticodeSignature -FilePath $FilePath
 }
 }
 
 ####################################################################################################
 function Select-Signed {
 <#.SYNOPSIS
    Select files based on the status of their Authenticode Signature.
 .DESCRIPTION
    The Select-Signed function filters files on the pipeline based on the state of their authenticode signature.
 .NOTES
    For information about Authenticode signatures in Windows PowerShell, type "get-help About_Signing".
 
    When specifying multiple values for a parameter, use commas to separate the values. For example, "-<parameter-name> <value1>, <value2>".
 .EXAMPLE
    ls *.ps1,*.ps[dm]1 | Select-Signed
    
    To get the signature information about the script.ps1 script file.
 .EXAMPLE
    ls *.ps1,*.psm1,*.psd1 | Get-AuthenticodeSignature
    
    Get the signature information for all the script and data files
 .EXAMPLE
    ls *.ps1,*.psm1,*.psd1 | Get-AuthenticodeSignature | Where {!(Test-Signature $_ -Valid)} | gci | Set-AuthenticodeSignature
 
    This command gets information about the Authenticode signature in all of the script and module files, and tests the signatures, then signs all of the ones that are not valid.
 .INPUTTYPE
    String. You can pipe the path to a file to Get-AuthenticodeSignature.
 .PARAMETER FilePath
    The path to the file being examined. Wildcards are permitted, but they must lead to a single file. The parameter name ("-FilePath") is optional.
    Aliases                      Path, FullName
 .RETURNVALUE
    System.Management.Automation.Signature
 ###################################################################################################>
 [CmdletBinding()]
 PARAM ( 
    [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true)]
    [Alias("FullName")]
    [ValidateScript({ 
       if((resolve-path $_).Provider.Name -ne "FileSystem") {
          throw "Specified Path is not in the FileSystem: '$_'" 
       }
       return $true
    })]
    [string[]]
    $FilePath
 ,
    [Parameter()]
    [switch]$MineOnly
 ,
    [Parameter()]
    [switch]$NotMineOnly
 ,
    [Parameter()]
    [Alias("HashMismatch")]
    [switch]$BrokenOnly
 ,
    [Parameter()]
    #
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
 PROCESS {
    if(!(Test-Path -PathType Leaf $FilePath)) { 
    } else {
 
       foreach($sig in Get-AuthenticodeSignature -FilePath $FilePath) {
       
       if($BrokenOnly   -and $sig.Status -ne "HashMismatch") 
       { 
          Write-Debug "$($sig.Status) - Not Broken: $FilePath"
          return 
       }
       
       if($ValidOnly    -and $sig.Status -ne "Valid") 
       { 
          Write-Debug "$($sig.Status) - Not Trusted: $FilePath"
          return 
       }
       
       if($TrustedOnly  -and (($sig.Status -ne "HashMismatch") -or !$sig.SignerCertificate) ) 
       { 
          Write-Debug "$($sig.Status) - Not Valid: $FilePath"
          return 
       }
       
       if($InvalidOnly  -and ($sig.Status -eq "Valid")) 
       { 
          Write-Debug "$($sig.Status) - Valid: $FilePath"
          return 
       }
       
       if($UnsignedOnly -and $sig.SignerCertificate ) 
       { 
          Write-Debug "$($sig.Status) - Signed: $FilePath"
          return 
       }
       
       if($MineOnly     -and (!($sig.SignerCertificate) -or ($sig.SignerCertificate.Thumbprint -ne $((Get-UserCertificate).Thumbprint))))
       {
          Write-Debug "Originally signed by someone else, thumbprint: $($sig.SignerCertificate.Thumbprint)"
          Write-Debug "Does not match your default certificate print: $((Get-UserCertificate).Thumbprint)"
          Write-Debug "     $FilePath"
          return 
       }
 
       if($NotMineOnly  -and (!($sig.SignerCertificate) -or ($sig.SignerCertificate.Thumbprint -eq $((Get-UserCertificate).Thumbprint))))
       {
          if($sig.SignerCertificate) {
             Write-Debug "Originally signed by you, thumbprint: $($sig.SignerCertificate.Thumbprint)"
             Write-Debug "Matches your default certificate print: $((Get-UserCertificate).Thumbprint)"
             Write-Debug "     $FilePath"
          }
          return 
       }
       
       if(!$BrokenOnly  -and !$TrustedOnly -and !$ValidOnly -and !$InvalidOnly -and !$UnsignedOnly -and !($sig.SignerCertificate) ) 
       { 
          Write-Debug "$($sig.Status) - Not Signed: $FilePath"
          return 
       }
       
       get-childItem $sig.Path
    }}
 }
 }
 Export-ModuleMember Set-AuthenticodeSignature, Get-AuthenticodeSignature, Test-Signature, Select-Signed, Get-UserCertificate
 
 
`

