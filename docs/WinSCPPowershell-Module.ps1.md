---
Author: battlechicken
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6057
Published Date: 2016-10-20t20
Archived Date: 2016-05-17t10
---

# winscppowershell module - 

## Description

in order to use this module, you need the winscp .net framework

## Comments



## Usage



## TODO



## script

`set-winscpdllfolder`

## Code

`#
 #
 $Script:WinSCPCSettingsOptionsObjectType = ""
 $Script:WinSCPDirHasBeenLoaded   = $false 
 
 Function Set-WinSCPDLLFolder {
 <#
 .Synopsis
    Specify the folder for WinSCP.exe and WinSCPnet.dll
 .DESCRIPTION
    Validates that WinSCP.exe and WinSCPnet.dll exists, then sets the folder containing WinSCP.exe and WinSCPnet.dll
 .PARAMETER DLLFolder
     String that contains the path containing the WinSCPnet.dll and WinSCPnet.dll
 .EXAMPLE
    Set-WinSCPDLLFolder -DLLFolder C:\Admin
    Sets the DLL folder for the WinSCPPowershell module to C:\Admin
 .NOTES
    The Function will throw an error if the executables do not exist EXACTLY as WinSCPnet.dll and WinSCP.exe in the target directory.  Once it gets set one time, no cmdlets require it to be provided.
 .COMPONENT
    WinSCPnet.dll
 .FUNCTIONALITY
    Sets the folder containing WinSCP.exe and WinSCPnet.dll
 #>
     param(
         [parameter(Mandatory=$true)]
         [string]$DLLFolder
     )
 
     if (test-path $DLLFolder){
         if ((Test-Path "${DLLFolder}\WinSCPnet.dll")){}
         else{
             throw "no WinSCPnet.dll found in ${DLLFolder}."
         }
         if ((Test-Path "${DLLFolder}\WinSCP.exe")){}
         else{
             throw "no WinSCP.exe found in ${DLLFolder}."
         }
     }
     else {throw "$DLLFolder does not appear to exist."}
     $script:WinSCPDLLFolder = $DLLFolder
     
     add-type -Path "$dllfolder\WinSCPnet.dll" -ErrorAction stop
     if ($?){
         $Script:WinSCPDirHasBeenLoaded = $true
     }
     $tempObj = New-Object WinSCP.SessionOptions
     $Script:WinSCPCSettingsOptionsObjectType = $tempObj.psobject.typenames[0]
 }
 
 Function Test-WinSCPRemotePathExists {
 <#
 .Synopsis
    Check if a remote file or folder exists on a SFTP site
 .DESCRIPTION
    Check if a remote file or folder exists on a SFTP site
 .PARAMETER WinSCPSessionObject
     WinSCP Session object.
 .PARAMETER RemotePath
     String containing a remote path
 .EXAMPLE
    Test-WinSCPRemotePathExists -WinSCPSessionObject $MySession -RemotePath /temp
    Checks if /temp exists on the SFTP site connected to $MySession
 .EXAMPLE
    $MySession | Test-WinSCPRemotePathExists -RemotePath /TextFile.txt
    Checks if /TextFile.txt exists on the SFTP site connected to $MySession
 .OUTPUTS
    Returns $True if $remotePath exists, $False if it does not.
 #>
         param(
 		[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[WinSCP.Session]$WinSCPSessionObject,
         
         [parameter(Mandatory=$true)]
         [string]$RemotePath
     )
     if ($RemotePath.contains("\")){
     }
     $WinSCPSessionObject.FileExists(($RemotePath.replace("*","")))
 }
 
 
 Function Get-WinSCPItemInfo {
 <#
 .Synopsis
    Returns a PSObject that contains information about $RemotePath
 .DESCRIPTION
    Returns a PSObject that contains File information data about the file or folder $RemotePath
 .EXAMPLE
    Get-WinSCPItemInfo -WinSCPSessionObject $MySession -RemotePath /temp
    Returns a PSObject that contains information about /temp
 .EXAMPLE
    $MySession | Get-WinSCPItemInfo -RemotePath /TextFile.txt
    Returns a PSObject that contains information about /TextFile.txt
 .OUTPUTS
    Outputs an object containing data like the following (if it exists) /temp in this example is a directory:
         Name            : /temp
         FileType        : D
         Length          : 0
         LastWriteTime   : 7/31/2014 1:22:10 PM
         FilePermissions : rwxrw-rw-
         IsDirectory     : True
    Othewise it returns nothing.
 #>    
     param(
 		[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[WinSCP.Session]$WinSCPSessionObject,
         
         [parameter(Mandatory=$true)]
         [string]$RemotePath
     )
     try{
         $WinSCPSessionObject.GetFileInfo($RemotePath)
     }
     catch {
         throw "$RemotePath does not exist"
     }
 }
 
 
 Function Get-PasswordFromEncryptedFile {
 <#
 .Synopsis
    Converts a password stored as a secure string to a file to plain text.
 .DESCRIPTION
    Converts a password stored as a secure string to a file to plain text.
 .EXAMPLE
    Get-PasswordFromEncryptedFile -PasswordFile "c:\admin\MyEncryptedPass.txt"
    Assuming the user who encryptedt he password is the same user executing the command, will decrypt the string in c:\admin\MyEncryptedPass.txt to plain-text.
 .OUTPUTS
    Outputs a string object
 .NOTES
    This function can be tricky.  it decrypts a securestring, so it will only be reversible by the same user that created the original encrypted file.  So, if my user is MyDomain\MyUsername, only MyDomain\MyUsername on the same machine can reverse the encryption.  Keep in mind the decrypt will only work if you created the file on that same machine.
 .FUNCTIONALITY
    Decrypts a secure string saved to a file.
 #>
     param(
         [parameter(Mandatory=$true)]
         [string]$PasswordFile
     )
 
     if (-not (Test-Path $PasswordFile)){
         throw "Nonexistent Password file"
     }
     else {
         try{
             $encryptedPass = get-content $PasswordFile | ConvertTo-SecureString
             $encryptedStr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($encryptedPass)
             [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($encryptedStr)
         }
         catch {
             throw "Error decrypting Secure string.  Only files encrypted by $env:USERNAME on $env:COMPUTERNAME can be decrypted in this session."
         }
 
     }
 }
 
 Function New-PasswordFile {
 <#
 .Synopsis
    Saves a string (a password most likely) to the specified file.
 .DESCRIPTION
    Saves a string (a password most likely) to the specified file.
 .EXAMPLE
    New-PasswordFile -PasswordFile c:\admin\MyEncryptedPassword.txt
    Prompts the user for a string, which gets saved encrypted to c:\admin\MyEncryptedPassword.txt
 #>
     param(
         [parameter(Mandatory=$true)]
         [string]$PasswordFile
     )
 
     read-host -AsSecureString "Enter a password" | ConvertFrom-SecureString -ErrorAction stop| out-file $PasswordFile -ErrorAction Stop
 }
 
 #
 #
 
 
 Function New-WinSCPBlankSessionOptions{
 <#
 .Synopsis
    Create a WinSCP session options object.
 .DESCRIPTION
    Create a winSCP session options object, which will be blank in order to create a more flexible sessionoptions object.
 .PARAMETER DLLFolder
     The path to the directory containing WinSCP.exe and WinSCPnet.dll.
 .EXAMPLE
    $myOptions = New-WinSCPBlankSessionOptions
    Creates a blank WinSCP Session options object and stores it to $myoptions.  You can then do $myOptions | Get-Member to show the available properties.
 .FUNCTIONALITY
    Create a blank WinSCP Session options object for connecting to SFTP.
 #>    
     param(
         [parameter()]
         [string]$DLLFolder=$Script:WinSCPDLLFolder
     )
 
 if (-not ($Script:WinSCPDirHasBeenLoaded)){
         Set-WinSCPDLLFolder -DLLFolder $DLLFolder
         $DLLFolder = $Script:WinSCPDLLFolder
     }
     
     New-Object WinSCP.SessionOptions
 
 
 }
 
 
 Function New-WinSCPSessionOptions {
 <#
 .Synopsis
    Create a WinSCP session options object.
 .DESCRIPTION
    Create a winSCP session options object, which contains all of the configuration options to connect to a site via SFTP.
 .PARAMETER Hostname
     The IP or name of the site to connect to
 .PARAMETER Port
     Used to specify a non-standard to connect to the specified Hostname.  If not specified, it will use the default protocol port.
 .PARAMETER Username
     Account to log into the SFTP/SCP/FTP site with
 .PARAMETER PasswordFile
     String path to an encrypted file containing a password to be read or created (depending on if -SetSecurePassword is selected)
 .PARAMETER SshHostKeyFingerprint
     String containing the ssh host key fingerprint.  Expects format like "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff"
 .PARAMETER SetSecurePassword
     Choose this if you want to create $PasswordFile in addition to reading $passwordFile in.  It will use read-host, so not for automated usage.
 .PARAMETER PlainTextPassword
     String containing a plain-text password to log into the SFTP/SCP/FTP site.
 .PARAMETER DLLFolder
     The path to the directory containing WinSCP.exe and WinSCPnet.dll.
 .EXAMPLE
    New-WinSCPSessionOptions -Hostname sftp.example.com -Username MyUsername -PasswordFile c:\admin\MyUsernamePass.txt -SshHostKeyFingerprint "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff"
    Creates a session options object pointing at sftp.example.com, which has a host key fingerprint of "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff". It will connect as MyUsername using the password encrypted in MyUsernamePass.txt
 .EXAMPLE
    New-WinSCPSessionOptions -Hostname sftp.example.com -Username MyUsername -PasswordFile c:\admin\MyUsernamePass.txt -SshHostKeyFingerprint "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff" -SetSecurePassword
    Creates a session options object pointing at sftp.example.com, which has a host key fingerprint of "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff". It will connect as MyUsername and will prompt the user to enter a string, which it will save as MyUsernamePass.txt
 .EXAMPLE
    New-WinSCPSessionOptions -Hostname sftp.example.com -Username MyUsername -PlainTextPassword "Password123" -SshHostKeyFingerprint "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff"
    Creates a session options object pointing at sftp.example.com, which has a host key fingerprint of "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff". It will connect as MyUsername using the password "Password123"
 .NOTES
    The function needs some work to increase the flexibiliy.  Right now it only works for SFTP since it is all the author had access to for testing.  to create a custom object, you can do New-Object WinSCP.SessionOptions.  This will hopefully change at some point in the future.
 .FUNCTIONALITY
    Create a WinSCP Session options object for connecting to SFTP.
 #>
     param(
         [parameter(Mandatory=$true)]
         [string]$Hostname,
         
         [parameter()]
         [string]$Protocol="Sftp",
 
         [parameter()]
 
         [parameter(Mandatory=$true)]
         [string]$Username,
     
         [parameter(Mandatory=$true,ParameterSetName="SecurePass")]
         [string]$PasswordFile,
 
         [parameter(Mandatory=$true)]
         [string]$SshHostKeyFingerprint,
 
         [parameter(ParameterSetName="SecurePass")]
         [switch]$SetSecurePassword,
 
         [parameter(Mandatory=$true,ParameterSetName="PlainTextPass")]
         [ValidateNotNullOrEmpty()]
         [string]$PlainTextPassword,
         
         [parameter()]
         [string]$DLLFolder=$Script:WinSCPDLLFolder
     )
     
     if (-not ($Script:WinSCPDirHasBeenLoaded)){
         Set-WinSCPDLLFolder -DLLFolder $DLLFolder
         $DLLFolder = $Script:WinSCPDLLFolder
     }
     
     $sessionOptions = New-Object WinSCP.SessionOptions
 
     if ($SetSecurePassword){
         New-PasswordFile -PasswordFile $PasswordFile
     }
 
     $sessionOptions.Protocol   = [WinSCP.Protocol]::$Protocol
     $sessionOptions.HostName   = $HostName
     $sessionOptions.UserName   = $Username
 
     if ($port){
         $sessionOptions.PortNumber = $Port
     }
 
     if ($PasswordFile){
         $sessionOptions.Password = Get-PasswordFromEncryptedFile -PasswordFile $PasswordFile
     }
     else {
         $sessionOptions.Password = $PlainTextPassword
     }
     $sessionOptions.SshHostKeyFingerprint = $SshHostKeyFingerprint
  
     $sessionOptions
 }
 
 #
 #
 Function Connect-WinSCPSFTPServer{
 <#
 .Synopsis
    Create a WinSCP session options object.
 .DESCRIPTION
    Create a winSCP session options object, which contains all of the configuration options to connect to a site via SFTP.
 .PARAMETER Hostname
     The IP or name of the site to connect to
 .PARAMETER Port
     Used to specify a non-standard to connect to the specified Hostname.  If not specified, it will use the default protocol port.
 .PARAMETER Username
     Account to log into the SFTP/SCP/FTP site with
 .PARAMETER PasswordFile
     String path to an encrypted file containing a password to be read or created (depending on if -SetSecurePassword is selected)
 .PARAMETER SshHostKeyFingerprint
     String containing the ssh host key fingerprint.  Expects format like "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff"
 .PARAMETER SetSecurePassword
     Choose this if you want to create $PasswordFile in addition to reading $passwordFile in.  It will use read-host, so not for automated usage.
 .PARAMETER PlainTextPassword
     String containing a plain-text password to log into the SFTP/SCP/FTP site.
 .PARAMETER DLLFolder
     The path to the directory containing WinSCP.exe and WinSCPnet.dll.
 .EXAMPLE
    Connect-WinSCPSFTPServer -Hostname sftp.example.com -Username MyUsername -PasswordFile c:\admin\MyUsernamePass.txt -SshHostKeyFingerprint "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff"
    Creates a session options object pointing at sftp.example.com, which has a host key fingerprint of "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff". It will connect as MyUsername using the password encrypted in MyUsernamePass.txt and connects to the server.
 .EXAMPLE
    Connect-WinSCPSFTPServer -Hostname sftp.example.com -Username MyUsername -PasswordFile c:\admin\MyUsernamePass.txt -SshHostKeyFingerprint "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff" -SetSecurePassword
    Creates a session options object pointing at sftp.example.com, which has a host key fingerprint of "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff". It will connect as MyUsername and will prompt the user to enter a string, which it will save as MyUsernamePass.txt and connects to the server.
 .EXAMPLE
    Connect-WinSCPSFTPServer -Hostname sftp.example.com -Username MyUsername -PlainTextPassword "Password123" -SshHostKeyFingerprint "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff"
    Creates a session options object pointing at sftp.example.com, which has a host key fingerprint of "ssh-rsa 2048 00:11:22:33:44:55:66:77:88:99:aa:bb:cc:dd:ee:ff". It will connect as MyUsername using the password "Password123" and connects to the server.
 .EXAMPLE
    Connect-WinSCPSFTPServer -WinSCPSessionOptionsObject $customSessionOptions
    Creates a session object connected to the specified server using the WinSCPSessionOptionsObject created prior to the connection.
 .NOTES
   
 .FUNCTIONALITY
    Create a WinSCP Session options object for connecting to SFTP then connects to the specified host.
 #>
     param(
         [parameter(Mandatory=$true,ParameterSetName="MakeObjPlainTextPass")]
         [parameter(Mandatory=$true,ParameterSetName="MakeObjSecurePass")]
         [string]$Hostname,
 
         [parameter(ParameterSetName="MakeObjPlainTextPass")]
         [parameter(ParameterSetName="MakeObjSecurePass")]
         [validateset("Ftp","Scp","Sftp")]
         [string]$Protocol="Sftp",
         
         [parameter(ParameterSetName="MakeObjPlainTextPass")]
         [parameter(ParameterSetName="MakeObjSecurePass")]
         [int]$Port = $null,
 
         [parameter(Mandatory=$true,ParameterSetName="MakeObjPlainTextPass")]
         [parameter(Mandatory=$true,ParameterSetName="MakeObjSecurePass")]
         [string]$Username,
     
         [parameter(Mandatory=$true,ParameterSetName="MakeObjSecurePass")]
         [string]$PasswordFile,
 
         [parameter(Mandatory=$true,ParameterSetName="MakeObjPlainTextPass")]
         [parameter(Mandatory=$true,ParameterSetName="MakeObjSecurePass")]
         [string]$SshHostKeyFingerprint,
 
         [parameter(Mandatory=$true,ParameterSetName="MakeObjPlainTextPass")]
         [ValidateNotNullOrEmpty()]
         [string]$PlainTextPassword,
 
         [parameter(ParameterSetName="PassObj")]
         [Object]$WinSCPSessionOptionsObject,
 
         [parameter()]
         [string]$DLLFolder=$Script:WinSCPDLLFolder,
 
         [parameter(ParameterSetName="MakeObjSecurePass")]
         [switch]$SetSecurePassword
     )
     
     if (-not ($Script:WinSCPDirHasBeenLoaded)){
         Set-WinSCPDLLFolder -DLLFolder $DLLFolder
         $DLLFolder = $Script:WinSCPDLLFolder
     }
 
     if ($SetSecurePassword){
         New-PasswordFile -PasswordFile $PasswordFile
     }
     
     $sessionOptions = $null
     if (-not $WinSCPSessionOptionsObject){
         if ($PasswordFile){
             $sessionOptions = New-WinSCPSessionOptions -DLLFolder $DLLFolder -Username $Username -Hostname $Hostname -Protocol $Protocol -Port $Port -PasswordFile $PasswordFile -SshHostKeyFingerprint $SshHostKeyFingerprint
         }
         if ($PlainTextPassword){
             $sessionOptions = New-WinSCPSessionOptions -DLLFolder $DLLFolder -Username $Username -Hostname $Hostname -Protocol $Protocol -Port $Port -PlainTextPassword $PlainTextPassword -SshHostKeyFingerprint $SshHostKeyFingerprint
         }
     }
     else{
         if ($WinSCPSessionOptionsObject.psobject.typenames[0] -eq $Script:WinSCPCSettingsOptionsObjectType){
             $sessionOptions = $WinSCPSessionOptionsObject
         }
         else{
             throw "invalid object type passed.  Object of type $Script:WinSCPCSettingsOptionsObjectType expected"
         }
     }
 
     $session = New-Object WinSCP.Session
 	$session.ExecutablePath = "$DLLFolder\WinSCP.exe"
 		
 	Write-Verbose "attempting to connect to server $Hostname"
 
 	try {
 	    $session.Open($sessionOptions)
 	}
     catch {
         throw "Exception opening session.  Double-check your credentials"
 	}
     if ($session.opened -eq $true){
         $session
     }
     else{
         throw "Unable to connect to server.  Check your connection details and try again"
     }
 }; New-Alias -Name Connect-WinSCPServer -Value Connect-WinSCPSFTPServer -Description "Connect to a MFT server"
 
 
 Function New-WinSCPTransferOptions {
 <#
 .Synopsis
    Creates a new WinSCP Transfer Options object
 .DESCRIPTION
    Long description
 .EXAMPLE
    Example of how to use this cmdlet
 .EXAMPLE
    Another example of how to use this cmdlet
 .INPUTS
    Inputs to this cmdlet (if any)
 .OUTPUTS
    Output from this cmdlet (if any)
 .NOTES
    General notes
 .COMPONENT
    The component this cmdlet belongs to
 .ROLE
    The role this cmdlet belongs to
 .FUNCTIONALITY
    The functionality that best describes this cmdlet
 #>
     param(
         [parameter()]
         [ValidateSet("Ascii","Automatic","Binary")]
         [string]$Mode="Binary",
 
         [parameter()]
         [ValidateSet("Default","Off","On","Smart")]
         [string]$ResumeSupport="Default",
 
         [parameter()]
         [string]$FileMask="",
 
         [parameter()]
         [ValidateScript({$_ -ge 0})]
         [int]$SpeedLimitKB=0
     )
     
 
 	$TransferOptions = New-Object WinSCP.TransferOptions
 	if($FileMask -ne "") {
 		$TransferOptions.FileMask = $FileMask
 	}
     $TransferOptions.SpeedLimit = $SpeedLimitKB
 	$TransferOptions.TransferMode = [WinSCP.TransferMode]::$Mode
     $TransferOptions.ResumeSupport.State = [WinSCP.TransferResumeSupportState]::$ResumeSupport
 
     $TransferOptions
 }
 
 	
 
 
 
 
 
 
 Function New-WinSCPTransfer {
 <#
 .Synopsis
    Specify the folder for WinSCP.exe and WinSCPnet.dll
 .DESCRIPTION
    Long description
 .EXAMPLE
    Example of how to use this cmdlet
 .EXAMPLE
    Another example of how to use this cmdlet
 .INPUTS
    Inputs to this cmdlet (if any)
 .OUTPUTS
    Output from this cmdlet (if any)
 .NOTES
    General notes
 .COMPONENT
    The component this cmdlet belongs to
 .ROLE
    The role this cmdlet belongs to
 .FUNCTIONALITY
    The functionality that best describes this cmdlet
 #>
     param(
 		[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[WinSCP.Session]$WinSCPSessionObject,
         
         [parameter()]
 		[validateset("Download","Upload")]
 		[string]$TransferType,
 
         [parameter(Mandatory=$true)]
         [string]$LocalPath,
 
         [parameter(Mandatory=$true)]
         [string]$RemotePath,
 
         [parameter()]
         [ValidateSet("Ascii","Automatic","Binary")]
         [string]$Mode="Binary",
 
         [parameter()]
         [string]$FileMask="",
 
         [parameter()]
         [switch]$DeleteSourceFilesAfterTransfer,
 
         [parameter()]
         [ValidateSet("Default","Off","On","Smart")]
         [string]$ResumeSupport="Default",
 
         [parameter()]
         [ValidateScript({$_ -ge 0})]
         [int]$SpeedLimitKB=0
     )
 
     if (-not (Test-WinSCPRemotePathExists -WinSCPSessionObject $WinSCPSessionObject -RemotePath ($RemotePath.replace("*","")))){
 		throw("The RemotePath does not exist on FTP: ${RemotePath}")
 	}
 
 	if ($DeleteSourceFilesAfterTransfer){
         $remove = $true
     }
     else {
         $remove = $false
     }
     
     #
     #
     switch ($TransferType.tolower()){
         "download" {
             #}
             break
         }
         "upload" {
             if (-not ($RemotePath.EndsWith("/")) -or ($RemotePath.EndsWith("*"))){
                 $RemotePath += "/"
             }
             break
         }
     }
 	
     $TransferOptions = New-WinSCPTransferOptions -Mode $Mode -ResumeSupport $ResumeSupport -FileMask $FileMask -SpeedLimitKB $SpeedLimitKB
 	
     switch ($TransferType.tolower()){
         upload {
 	        Write-Verbose "Beginning upload..."
 	        $result = $WinSCPSessionObject.PutFiles($LocalPath, $RemotePath, $remove, $TransferOptions)
         }
         download{
 		    Write-Verbose "Beginning download..."
 		    $result = $WinSCPSessionObject.GetFiles($RemotePath, $LocalPath, $remove, $TransferOptions)
         }
     }
 
 	$result.Check()
 		
 
 
 	if(($result.Transfers | Measure).count -gt 0) {
 		write-verbose "Files successfully transfered:"
         $i = 1
 		$result.Transfers | % {
 			Write-Verbose "`t$i - $($_.Destination)"
             $i++
 		}
 	}
 	if(($result.Failures | Measure).count -gt 0) {
 		Write-Verbose "Failed file transfers:"
         $i=1
 		$result.Failures | % {
 			Write-Verbose "`t$i - $($_.FileName)"
             $i++
 		}
 	}
 	if(-not $result.IsSuccess) {
 		throw("FTP transfer Failed")
 	}
 }
 
 
 #
 #
 #
 #
 
 Function New-WinSCPUpload {
 <#
 .Synopsis
    Specify the folder for WinSCP.exe and WinSCPnet.dll
 .DESCRIPTION
    Long description
 .EXAMPLE
    Example of how to use this cmdlet
 .EXAMPLE
    Another example of how to use this cmdlet
 .INPUTS
    Inputs to this cmdlet (if any)
 .OUTPUTS
    Output from this cmdlet (if any)
 .NOTES
    General notes
 .COMPONENT
    The component this cmdlet belongs to
 .ROLE
    The role this cmdlet belongs to
 .FUNCTIONALITY
    The functionality that best describes this cmdlet
 #>
     param(
 		[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[WinSCP.Session]$WinSCPSessionObject,
         
         [parameter(Mandatory=$true)]
         [string]$LocalPath,
 
         [parameter(Mandatory=$true)]
         [string]$RemotePath,
 
         [parameter()]
         [ValidateSet("Ascii","Automatic","Binary")]
         [string]$Mode="Binary",
 
         [parameter()]
         [string]$FileMask="",
 
         [parameter()]
         [switch]$DeleteSourceFilesAfterTransfer,
 
         [parameter()]
         [ValidateSet("Default","Off","On","Smart")]
         [string]$ResumeSupport="Default",
 
         [parameter()]
         [ValidateScript({$_ -ge 0})]
         [int]$SpeedLimitKB=0
     )
 
     if ($DeleteSourceFilesAfterTransfer){
         $WinSCPSessionObject | New-WinSCPTransfer -TransferType Upload -LocalPath $LocalPath -RemotePath $RemotePath -Mode $Mode -FileMask $FileMask -ResumeSupport $ResumeSupport -SpeedLimitKB $SpeedLimitKB -DeleteSourceFilesAfterTransfer
     }
     else{
         $WinSCPSessionObject | New-WinSCPTransfer -TransferType Upload -LocalPath $LocalPath -RemotePath $RemotePath -Mode $Mode -FileMask $FileMask -ResumeSupport $ResumeSupport -SpeedLimitKB $SpeedLimitKB
     }
     
 }; New-Alias -Name Upload-Files -Value New-WinSCPUpload -Description "Upload a file using WinSCP"
 
 #
 #
 #
 #
 
 Function New-WinSCPDownload {
 <#
 .Synopsis
    Specify the folder for WinSCP.exe and WinSCPnet.dll
 .DESCRIPTION
    Long description
 .EXAMPLE
    Example of how to use this cmdlet
 .EXAMPLE
    Another example of how to use this cmdlet
 .INPUTS
    Inputs to this cmdlet (if any)
 .OUTPUTS
    Output from this cmdlet (if any)
 .NOTES
    General notes
 .COMPONENT
    The component this cmdlet belongs to
 .ROLE
    The role this cmdlet belongs to
 .FUNCTIONALITY
    The functionality that best describes this cmdlet
 #>
     param(
 		[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[WinSCP.Session]$WinSCPSessionObject=$null,
         
         [parameter(Mandatory=$true)]
         [string]$LocalPath,
 
         [parameter(Mandatory=$true)]
         [string]$RemotePath,
 
         [parameter()]
         [ValidateSet("Ascii","Automatic","Binary")]
         [string]$Mode="Binary",
 
         [parameter()]
         [string]$FileMask="",
 
         [parameter()]
         [switch]$DeleteSourceFilesAfterTransfer,
 
         [parameter()]
         [ValidateSet("Default","Off","On","Smart")]
         [string]$ResumeSupport="Default",
 
         [parameter()]
         [ValidateScript({$_ -ge 0})]
         [int]$SpeedLimitKB=0
     )
 
     if ($DeleteSourceFilesAfterTransfer){
         $WinSCPSessionObject | New-WinSCPTransfer -TransferType Download -LocalPath $LocalPath -RemotePath $RemotePath -Mode $Mode -FileMask $FileMask -ResumeSupport $ResumeSupport -SpeedLimitKB $SpeedLimitKB -DeleteSourceFilesAfterTransfer
     }
     else{
         $WinSCPSessionObject | New-WinSCPTransfer -TransferType Download -LocalPath $LocalPath -RemotePath $RemotePath -Mode $Mode -FileMask $FileMask -ResumeSupport $ResumeSupport -SpeedLimitKB $SpeedLimitKB
     }
     
 }; New-Alias -Name Download-Files -Value New-WinSCPDownload -Description "Upload a file using WinSCP"
 
 
 Function Close-WinSCPSFTPServerConnection {
 <#
 .Synopsis
    Specify the folder for WinSCP.exe and WinSCPnet.dll
 .DESCRIPTION
    Long description
 .EXAMPLE
    Example of how to use this cmdlet
 .EXAMPLE
    Another example of how to use this cmdlet
 .INPUTS
    Inputs to this cmdlet (if any)
 .OUTPUTS
    Output from this cmdlet (if any)
 .NOTES
    General notes
 .COMPONENT
    The component this cmdlet belongs to
 .ROLE
    The role this cmdlet belongs to
 .FUNCTIONALITY
    The functionality that best describes this cmdlet
 #>
     param(
 		[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[WinSCP.Session]$WinSCPSessionObject=$null
     )
 
     $WinSCPSessionObject.dispose()
 }
 
 
 
 
 #
 #
 Function Get-WinSCPDirectoryList {
 <#
 .Synopsis
    Specify the folder for WinSCP.exe and WinSCPnet.dll
 .DESCRIPTION
    Long description
 .EXAMPLE
    Example of how to use this cmdlet
 .EXAMPLE
    Another example of how to use this cmdlet
 .INPUTS
    Inputs to this cmdlet (if any)
 .OUTPUTS
    Output from this cmdlet (if any)
 .NOTES
    General notes
 .COMPONENT
    The component this cmdlet belongs to
 .ROLE
    The role this cmdlet belongs to
 .FUNCTIONALITY
    The functionality that best describes this cmdlet
 #>
     param(
 		[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[WinSCP.Session]$WinSCPSessionObject=$null,
         
         [parameter(Mandatory=$true)]
         [string]$RemotePath
     )
 
     if (Test-WinSCPRemotePathExists -WinSCPSessionObject $WinSCPSessionObject -RemotePath $RemotePath){
         $WinSCPSessionObject.ListDirectory($RemotePath) | select -ExpandProperty files
     }
     else {
         Write-Warning "$RemotePath Does not exist"
         return $null
     }
 }
 
 
 #
 #
 Function New-WinSCPRemoteDirectory {
 <#
 .Synopsis
    Specify the folder for WinSCP.exe and WinSCPnet.dll
 .DESCRIPTION
    Long description
 .EXAMPLE
    Example of how to use this cmdlet
 .EXAMPLE
    Another example of how to use this cmdlet
 .INPUTS
    Inputs to this cmdlet (if any)
 .OUTPUTS
    Output from this cmdlet (if any)
 .NOTES
    General notes
 .COMPONENT
    The component this cmdlet belongs to
 .ROLE
    The role this cmdlet belongs to
 .FUNCTIONALITY
    The functionality that best describes this cmdlet
 #>
     param(
 		[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[WinSCP.Session]$WinSCPSessionObject,
         
         [parameter(Mandatory=$true)]
         [string]$RemotePath
     )
 
     if (Test-WinSCPRemotePathExists -WinSCPSessionObject $WinSCPSessionObject -RemotePath $RemotePath){
         Write-Verbose "$RemotePath already existed"
     }
     else{
         try {
             $WinSCPSessionObject.CreateDirectory($RemotePath)
             if ($?){
                 Write-Verbose "$RemotePath has been created"
             }
         }
         catch {
             throw "Error creating $RemotePath"
         }
     }
 }
 
 
 
 Function Remove-WinSCPRemoteItem {
 <#
 .Synopsis
    Specify the folder for WinSCP.exe and WinSCPnet.dll
 .DESCRIPTION
    Long description
 .EXAMPLE
    Example of how to use this cmdlet
 .EXAMPLE
    Another example of how to use this cmdlet
 .INPUTS
    Inputs to this cmdlet (if any)
 .OUTPUTS
    Output from this cmdlet (if any)
 .NOTES
    General notes
 .COMPONENT
    The component this cmdlet belongs to
 .ROLE
    The role this cmdlet belongs to
 .FUNCTIONALITY
    The functionality that best describes this cmdlet
 #>
     param(
 		[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[WinSCP.Session]$WinSCPSessionObject,
         
         [parameter(Mandatory=$true)]
         [string]$RemotePath,
 
         [parameter()]
         [switch]$force
     )
     $delData = $null
     $dirContents = $null
 
     if (-not (Test-WinSCPRemotePathExists -WinSCPSessionObject $WinSCPSessionObject -RemotePath $RemotePath)){
         Write-Verbose "$RemotePath does not exist.  No action taken."
     }
     else{
         $itemInfo = Get-WinSCPItemInfo -WinSCPSessionObject $WinSCPSessionObject -RemotePath $RemotePath
         if ($itemInfo.isDirectory){
             $dirContents = @(Get-WinSCPDirectoryList -WinSCPSessionObject $WinSCPSessionObject -RemotePath $RemotePath)
             if (($dirContents.count -gt 0) -and (-not ($force))){
                 Write-Warning "${RemotePath} is a directory with additional files in it. at least $($dirContents.count) items. This check does NOT recurse - there may be a lot more files."
                 $input = (read-host "Are you sure you want to delete everything inside ${RemotePath}?").tolower()
                 if ($input.StartsWith("y")){
                     $delData = $WinSCPSessionObject.RemoveFiles($RemotePath)
                 }
                 else {
                     Write-Verbose "$RemotePath has not been deleted.  No action taken"
                 }
             }
             else {
                 $delData = $WinSCPSessionObject.RemoveFiles($RemotePath)
             }
         }
         else{
             $delData = $WinSCPSessionObject.RemoveFiles($RemotePath)
         }
     }
 
     #
     #
     if ($delData -ne $null){
 	    if(($delData.Removals | Measure).count -gt 0) {
 		    write-verbose "Files/folders successfully deleted:"
             $i = 1
 		    $delData.Removals | % {
 			    Write-Verbose "$i - $($_.FileName)"
                 $i++
 		    }
 	    }
 	    if(($delData.Failures | Measure).count -gt 0) {
 		    Write-Verbose "Failed file/folder deletions:"
             $i=1
 		    $delData.Failures | % {
 			    Write-Verbose "$i - $($_.FileName)"
                 $i++
 		    }
 	    }
 	    if(-not $delData.IsSuccess) {
 		    throw("Deletion of ${RemotePath} failed.")
 	    }
 
 
     }
 }
 
 
 function Sync-WinSCPDirectory{
     param(
     	[parameter(ValueFromPipeLine=$true,Mandatory=$true)]
 		[WinSCP.Session]$WinSCPSessionObject,
 
         [parameter(Mandatory=$true)]
         [string]$LocalPath,
 
         [parameter(Mandatory=$true)]
         [string]$RemotePath,
 
         [Parameter(Mandatory=$true)]
         [ValidateSet("Local","Remote","Both")]
         [WinSCP.SynchronizationMode]$SynchronizationMode,
 
         [Parameter()]
         [ValidateSet("None","Time","Size","Either")]
         [WinSCP.SynchronizationCriteria]$SynchronizationCriteria="Time",
 
         [Parameter()]
         [Switch]$Mirror,
 
         [parameter()]
         [ValidateSet("Ascii","Automatic","Binary")]
         [string]$TransferMode="Binary",
 
         [parameter()]
         [string]$FileMask="",
 
         [Parameter()]
         [Switch]$RemoveFilesAfterTransfer,
 
         [parameter()]
         [ValidateSet("Default","Off","On","Smart")]
         [string]$ResumeSupport="Default",
 
         [parameter()]
         [ValidateScript({$_ -ge 0})]
         [int]$SpeedLimitKB=0
     )
 
     $TransferOptions = New-WinSCPTransferOptions -Mode $Mode -ResumeSupport $ResumeSupport -FileMask $FileMask -SpeedLimitKB $SpeedLimitKB
 
     try{
         $output = $WinSCPSessionObject.SynchronizeDirectories($SynchronizationMode, $LocalPath, $RemotePath, $RemoveFilesAfterTransfer.IsPresent, $Mirror.IsPresent, $SynchronizationCriteria, $TransferOptions)
     }
     catch {
         Throw $_
     }
     
     if (($output.uploads | Measure).count -gt 0){
         write-verbose "Files/folders successfully uploaded:"
         $i = 1
 		$output.uploads | % {Write-Verbose "`t$i - $($_.FileName)"; $i++}
     }
     
     if (($output.downloads | Measure).count -gt 0){
         write-verbose "Files/folders successfully downloaded:"
         $i = 1
 		$output.downloads | % {Write-Verbose "`t$i - $($_.FileName)"; $i++}
     }
     if (($output.Failures | Measure).count -gt 0){
         write-verbose "Files/folders Failed:"
         $i = 1
 		$output.Failures | % {Write-Verbose "`t$i - $($_.FileName)"; $i++}
     }
 }
 
 
 #
 #
 $modulePath = split-path $SCRIPT:MyInvocation.MyCommand.Path -parent
 if ((Test-Path "$modulePath\WinSCP.exe") -and (Test-Path "$modulePath\WinSCPnet.dll")){
     Set-WinSCPDLLFolder -DLLFolder $modulePath
 }
 else{
     throw "Unable to find the WinSCP executables WinSCPnet.dll and WinSCP.exe in $modulePath"
 }
 #
 
 
 export-modulemember -alias * -function *
`

