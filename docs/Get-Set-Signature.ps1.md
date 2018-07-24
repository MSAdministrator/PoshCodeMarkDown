---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 2.5
Encoding: ascii
License: cc0
PoshCode ID: 1966
Published Date: 2011-07-09t21
Archived Date: 2014-05-16t04
---

# get/set signature - 

## Description

the improved code-signing module with automatic cert picking and finding.

## Comments



## Usage



## TODO



## script

`convertto-stringdata`

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
 
 
 function ConvertTo-StringData { param([Parameter(ValueFromPipeline=$true)]$InputObject)
    switch($InputObject.GetType().FullName) {
       'System.Collections.Hashtable' { ($InputObject.Keys | % { "$_=$($InputObject.$_)" }) -join "`n" }
 }  }
 
 function Get-UserCertificate {
 <#
 .SYNOPSIS
  Gets the user's default signing certificate so we don't have to ask them over and over...
 .DESCRIPTION
  The Get-UserCertificate function retrieves and returns a certificate from the user. It also stores the certificate so it can be reused without re-querying for the location and/or password ... 
 .RETURNVALUE
  An X509Certificate2 suitable for code-signing
 #>
 [CmdletBinding()]
 param ( $Name )
 begin {
    if($Name) { 
       $Script:UserCertificate = Get-AuthenticodeCertificate $Name
    }
 }
 end {
    $PrivateData = ConvertFrom-StringData (Get-Module -ListAvailable $PSCmdlet.MyInvocation.MyCommand.Module.Name).PrivateData
    if(!$PrivateData) { $PrivateData = @{} }
    
    if(!(Test-Path Variable:Script:UserCertificate) -or 
       $Script:UserCertificate -isnot [System.Security.Cryptography.X509Certificates.X509Certificate2] -or
       $Script:UserCertificate.Thumbprint -ne $PrivateData.${Env:ComputerName}
    ) {
       if($VerbosePreference -gt "SilentlyContinue") {
          if(!(Test-Path Variable:Script:UserCertificate)) {
             Write-Verbose "Loading User Certificate from Module Config: $($PrivateData.${Env:ComputerName} )"
          } else {
             Write-Verbose "Saving User Certificate to Module Config: ($($Script:UserCertificate.Thumbprint) -ne $($PrivateData.${Env:ComputerName}))"
          }
       }
       
       Write-Debug "PrivateData: $($ExecutionContext.SessionState.Module | fl * | Out-String)"
       if(!(Test-Path Variable:Script:UserCertificate) -or $Script:UserCertificate -isnot [System.Security.Cryptography.X509Certificates.X509Certificate2]) {
          $Script:UserCertificate = Get-AuthenticodeCertificate $PrivateData.${Env:ComputerName}
       }
       Write-Verbose "Confirming Certificate: $($Script:UserCertificate.Thumbprint)"
       
       if($Script:UserCertificate -and (!$PrivateData.${Env:ComputerName} -or
                                       ($PrivateData.${Env:ComputerName} -ne $Script:UserCertificate.Thumbprint)))
       {
          $PrivateData.${Env:ComputerName} = $Script:UserCertificate.Thumbprint
          
          Write-Verbose "Updating Module Metadata"
          if($Host.UI -and $Host.UI.PromptForChoice -and (0 -eq
             $Host.UI.PromptForChoice("Keep this certificate for future sessions?", $Script:UserCertificate,
             [Management.Automation.Host.ChoiceDescription[]]@("&Yes","&No"), 0))
          ) {
             $mVersion = (Get-Module -ListAvailable $PSCmdlet.MyInvocation.MyCommand.Module.Name).Version
             Write-Verbose "Version: $mVersion"
             if($MVersion -le "2.5") { $MVersion = 2.5 }
             
             New-ModuleManifest $PSScriptRoot\Authenticode.psd1                         `
                        -ModuleToProcess Authenticode.psm1                `
                        -Author 'Joel Bennett' -Company 'HuddledMasses.org'             `
                        -ModuleVersion  $MVersion                                       `
                        -PowerShellVersion '2.0'                                        `
                        -Copyright '(c) 2008-2010, Joel Bennett'                        `
                        -Desc 'Function wrappers for Authenticode Signing cmdlets'      `
                        -Types @() -Formats @() -RequiredModules @()                    `
                        -RequiredAssemblies @() -FileList @() -NestedModules @()        `
                        -PrivateData ($PrivateData | ConvertTo-StringData)
             $null = Sign $PSScriptRoot\Authenticode.psd1 -Cert $Script:UserCertificate
          }
       }
    }
    return $Script:UserCertificate
 }
 }
 
 function Get-AuthenticodeCertificate {
 [CmdletBinding()]
 param (
    $Name = $(Get-UserCertificate)
 )
 
 end {
    $Certificate = $Name
    while($Certificate -isnot [System.Security.Cryptography.X509Certificates.X509Certificate2]) {
       trap {
          Write-Host "The authenticode module requires a code-signing certificate, and can't find yours!"
          Write-Host
          Write-Host "If this is the first time you've seen this error, please run Get-AuthenticodeCertificate by hand and specify the full path to your PFX file, or the Thumbprint of a cert in your OS Cert store -- and then answer YES to save that cert in the 'PrivateData' of the Authenticode Module metadata."
          Write-Host
          Write-Host "If you have seen this error multiple times, you may need to manually create a module manifest for this module with the path to your cert, and/or specify the certificate name each time you use it."
          Write-Error $_
          continue      
       }
       if(!$Name) {
          $certs = @(Get-ChildItem Cert:\ -Recurse -CodeSign | Sort NotAfter)
          if($certs.Count) {
             Write-Host "You have $($certs.Count) code signing certificates in your local certificate storage which you can specify by partial Thumbprint, or you may specify the path to a .pfx file:" -fore cyan
             $certs | Out-Host
          }
          $Name = $(Read-Host "Please specify a user certificate (wildcards allowed)")
          if(!$Name) { return }
       }
       
       Write-Verbose "Certificate Path: $Name"
       $ResolvedPath = Get-ChildItem Cert:\CurrentUser\My -Recurse -CodeSign | Where {$_.ThumbPrint -like $Name } | Select -Expand PSPath
       if(!$ResolvedPath) {
          $ResolvedPath = Get-ChildItem Cert:\ -Recurse -CodeSign | Where {$_.ThumbPrint -like $Name } | Select -Expand PSPath
       }
       
       if(!$ResolvedPath) {
          Write-Verbose "Not a Certificate path: $Path"
          $ResolvedPath = Resolve-Path $Name -ErrorAction "SilentlyContinue" | Where { Test-Path $_ -PathType Leaf -ErrorAction "SilentlyContinue" }
       }
       
       if(!$ResolvedPath) {
          Write-Verbose "Not a full or legit relative path Path: $ResolvedPath"
          $ResolvedPath = Resolve-Path (Join-Path $PsScriptRoot $Name -ErrorAction "SilentlyContinue") -ErrorAction "SilentlyContinue" | Where { Test-Path $_ -PathType Leaf -ErrorAction "SilentlyContinue" }
          Write-Verbose "Resolved File Path: $ResolvedPath"
       }
       
       if(@($ResolvedPath).Count -gt 1) {
          throw "You need to specify enough of the name to narrow it to a single certificate. '$Name' returned $(@($ResolvedPath).Count):`n$($ResolvedPath|Out-String)"
       }
 
       $Certificate = get-item $ResolvedPath -ErrorAction "SilentlyContinue"
       if($Certificate -is [System.IO.FileInfo]) {
          $Certificate = Get-PfxCertificate $Certificate -ErrorAction "SilentlyContinue"
       }
    }
    Write-Verbose "Certificate: $($Certificate | Out-String)"
    return $Certificate
 }
 }
 
 function Test-AuthenticodeSignature {
 <#
 .SYNOPSIS
    Tests a script signature to see if it is valid, or at least unaltered.
 .DESCRIPTION
    The Test-AuthenticodeSignature function processes the output of Get-AuthenticodeSignature to determine if it 
    is Valid, OR **unaltered** and signed by the user's certificate
 .PARAMETER Signature
    Specifies the signature object to test. This should be the output of Get-AuthenticodeSignature.
 .PARAMETER ForceValid
    Switch parameter, forces the signature to be valid -- otherwise, even if the certificate chain can't be verified, we will accept the cert which matches the "user" certificate (see Get-UserCertificate).
    Aliases                      Valid
 .EXAMPLE
    ls *.ps1 | Get-AuthenticodeSignature | Where {Test-AuthenticodeSignature $_}
    To get the signature reports for all the scripts that we consider safely signed.
 .EXAMPLE
    ls | ? { gas $_ | Test-AuthenticodeSignature }
    List all the valid signed scripts (or scripts signed by our cert)
 .NOTES
    Test-AuthenticodeSignature returns TRUE even if the root CA certificate can't be verified, as long as the signing certificate's thumbnail matches the one specified by Get-UserCertificate.
 .INPUTTYPE
    System.Management.Automation.Signature
 .RETURNVALUE
    Boolean value representing whether the script's signature is valid, or YOUR certificate
 #>
 [CmdletBinding()]
 param (
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
    ls *.ps1,*.psm1,*.psd1 | Get-AuthenticodeSignature | Where {!(Test-AuthenticodeSignature $_ -Valid)} | gci | Set-AuthenticodeSignature
 
    List all the script files, and get and test their signatures, and then sign all of the ones that are not valid, using the user's default certificate.
 .EXAMPLE
    Set-AuthenticodeSignature -Module PSCX
    
    Signs the whole PSCX module at once (all the ps1, psm1, psd1, dll, exe, and ps1xml files, etc.).
 .INPUTTYPE
    String. You can pipe a file path to Set-AuthenticodeSignature.
 .PARAMETER FilePath
    Specifies the path to a file that is being signed.
    Aliases                      Path, FullName
 .PARAMETER ModuleName
    Specifies a module name (or path) to sign. 
    
    When you specify a module name, all of the files in that folder and it's subfolders are signed (if they're signable).   
 .PARAMETER Certificate
    Specifies the certificate that will be used to sign the script or file. Enter a variable that stores an object representing the certificate or an expression that gets the certificate.
   
    To find a certificate, use Get-PfxCertificate or use the Get-ChildItem cmdlet in the Certificate (Cert:) drive. If the certificate is not valid or does not have code-signing authority, the command fails.
 .PARAMETER Force
    Allows the cmdlet to append a signature to a read-only file. Even using the Force parameter, the cmdlet cannot override security restrictions.
 .Parameter HashAlgorithm
    Specifies the hashing algorithm that Windows uses to compute the digital signature for the file. The default is SHA1, which is the Windows default hashing algorithm.
 
    Files that are signed with a different hashing algorithm might not be recognized on other systems.
 .PARAMETER IncludeChain
    Determines which certificates in the certificate trust chain are included in the digital signature. "NotRoot" is the default.
 
    Valid values are:
 
    -- Signer: Includes only the signer's certificate.
 
    -- NotRoot: Includes all of the certificates in the certificate chain, except for the root authority.
 
    --All: Includes all the certificates in the certificate chain.
 
 .PARAMETER TimestampServer
    Uses the specified time stamp server to add a time stamp to the signature. Type the URL of the time stamp server as a string.
    Defaults to Verisign's server: http://timestamp.verisign.com/scripts/timstamp.dll
 
    The time stamp represents the exact time that the certificate was added to the file. A time stamp prevents the script from failing if the certificate expires because users and programs can verify that the certificate was valid atthe time of signing.
 .RETURNVALUE
    System.Management.Automation.Signature
 ###################################################################################################>
 [CmdletBinding(DefaultParameterSetName="File")]
 param ( 
    [Parameter(Position=1, Mandatory=$true, ValueFromPipelineByPropertyName=$true, ValueFromPipeline=$true, ParameterSetName="File")]
    [Alias("FullName","Path")]
    [ValidateScript({ 
       if((resolve-path $_).Provider.Name -ne "FileSystem") {
          throw "Specified Path is not in the FileSystem: '$_'" 
       }
       return $true
    })]
    [string[]]$FilePath
 ,
    [Parameter(Position=1, Mandatory=$true, ParameterSetName="Module")]
    [ValidateScript({ 
       Write-Verbose $_
       if(!(Get-Module -List $_)) {
          throw "Cannot find a module by that name: '$_'" 
       }
       return $true
    })]
    [string[]]$ModuleName
 , 
    [Parameter(Position=2, Mandatory=$false)]
    $Certificate = $(Get-UserCertificate)
 , 
    [Switch]$Force
 ,
    [ValidateSet("SHA","MD5","SHA1","SHA256","SHA384","SHA512")]
    [String]$HashAlgorithm #="SHA1"
 ,
    [ValidateSet("Signer","NotRoot","All")]
    [String]$IncludeChain #="NotRoot"
 ,
    [String]$TimestampServer = "http://timestamp.verisign.com/scripts/timstamp.dll"
 )
 
 begin {
    if($PSCmdlet.ParameterSetName -eq "Module"){
       Write-Verbose ($ModuleName -join ", ")
       $FilePath = Get-Module -List $ModuleName | Split-Path | Get-ChildItem -Recurse | Where { !$_.PsIsContainer } | Select -Expand FullName
       $FilePath | Write-Verbose 
       $null = $PSBoundParameters.Remove("ModuleName")
    }
    if($Certificate -isnot [System.Security.Cryptography.X509Certificates.X509Certificate2]) {
       $Certificate = Get-AuthenticodeCertificate $Certificate
    }
    $PSBoundParameters["Certificate"] = $Certificate
 }
 process {
    Write-Verbose "Set Authenticode Signature on $FilePath with $($Certificate | Out-String)"
    $PSBoundParameters["FilePath"] = $FilePath = $(Resolve-Path $FilePath)
    if(Test-Path $FilePath -Type Leaf) {
       Microsoft.PowerShell.Security\Set-AuthenticodeSignature @PSBoundParameters
    } else {
       Write-Warning "Cannot sign folders: '$FilePath'" 
    }
 }
 }
 
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
    ls *.ps1,*.psm1,*.psd1 | Get-AuthenticodeSignature | Where {!(Test-AuthenticodeSignature $_ -Valid)} | gci | Set-AuthenticodeSignature
 
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
 param ( 
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
 
 process {
    Microsoft.PowerShell.Security\Get-AuthenticodeSignature -FilePath $FilePath
 }
 }
 
 ####################################################################################################
 function Select-AuthenticodeSigned {
 <#.SYNOPSIS
    Select files based on the status of their Authenticode Signature.
 .DESCRIPTION
    The Select-AuthenticodeSigned function filters files on the pipeline based on the state of their authenticode signature.
 .NOTES
    For information about Authenticode signatures in Windows PowerShell, type "get-help About_Signing".
 
    When specifying multiple values for a parameter, use commas to separate the values. For example, "-<parameter-name> <value1>, <value2>".
 .EXAMPLE
    ls *.ps1,*.ps[dm]1 | Select-AuthenticodeSigned
    
    To get the signature information about the script.ps1 script file.
 .EXAMPLE
    ls *.ps1,*.psm1,*.psd1 | Get-AuthenticodeSignature
    
    Get the signature information for all the script and data files
 .EXAMPLE
    ls *.ps1,*.psm1,*.psd1 | Get-AuthenticodeSignature | Where {!(Test-AuthenticodeSignature $_ -Valid)} | gci | Set-AuthenticodeSignature
 
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
 param ( 
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
 process {
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
 
 
 function Start-AutoSign {
 param($Path=".", $Filter= "*.ps*", [Switch]$Recurse, $CertPath, [Switch]$NoNotify)
 
 if(!$NoNotify -and (Get-Module Growl -ListAvailable -ErrorAction 0)) {
    Import-Module Growl
    Register-GrowlType AutoSign "Signing File" -ErrorAction 0
 } else { $NoNotify = $false }
 
 $realItem = Get-Item $Path -ErrorAction Stop
 if (-not $realItem) { return } 
 
 $Action = {
    $InvalidForm = "The form specified for the subject is not one supported or known by the specified trust provider"
    
    ForEach($file in Get-ChildItem $eventArgs.FullPath | Get-AuthenticodeSignature | 
       Where-Object { $_.Status -ne "Valid" -and $_.StatusMessage -ne $invalidForm } | 
       Select-Object -ExpandProperty Path ) 
    {
       if(!$NoNotify) {
          Send-Growl AutoSign "Signing File" "File $($eventArgs.ChangeType), signing:" "$file"
       }
       if($CertPath) {
          Set-AuthenticodeSignature -FilePath $file -Certificate $CertPath
       } else {
          Set-AuthenticodeSignature -FilePath $file
       }
    }
 }
 $watcher = New-Object IO.FileSystemWatcher $realItem.Fullname, $filter -Property @{ IncludeSubdirectories = $Recurse }
 Register-ObjectEvent $watcher "Created" "AutoSignCreated$($realItem.Fullname)" -Action $Action > $null
 Register-ObjectEvent $watcher "Changed" "AutoSignChanged$($realItem.Fullname)" -Action $Action > $null
 Register-ObjectEvent $watcher "Renamed" "AutoSignChanged$($realItem.Fullname)" -Action $Action > $null
 
 }
 
 Set-Alias gas          Get-AuthenticodeSignature -Description "Authenticode Module Alias"
 Set-Alias sas          Set-AuthenticodeSignature -Description "Authenticode Module Alias"
 Set-Alias slas         Select-AuthenticodeSigned -Description "Authenticode Module Alias"
 Set-Alias sign         Set-AuthenticodeSignature -Description "Authenticode Module Alias"
 
 Export-ModuleMember -Alias gas,sas,slas,sign -Function Set-AuthenticodeSignature, Get-AuthenticodeSignature, Test-AuthenticodeSignature, Select-AuthenticodeSigned, Get-UserCertificate, Get-AuthenticodeCertificate, Start-AutoSign
 
`

