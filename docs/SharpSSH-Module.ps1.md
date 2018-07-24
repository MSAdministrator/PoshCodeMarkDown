---
Author: tysonkopczynski
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 979
Published Date: 2011-03-29t04
Archived Date: 2011-12-30t05
---

# sharpssh module - 

## Description

as promised, i started to expand on joel’s functions (http

## Comments



## Usage



## TODO



## function

`new-sharpsession`

## Code

`#
 #
 ##################################################
 ##################################################
 #-------------------------------------------------
 #-------------------------------------------------
 #-------------------------------------------------
 function New-SharpSession {
     .Synopsis
     	Used to open an SSH or SCP Session to the specified server.
     .Description
     	Uses SharpSSH to open an SSH or SCP Session to the specified server.
     .Parameter Host
     	The hostname or IP address that you want to connect to.
     .Parameter UserName
     	The username string used for the connection.
     .Parameter Key
     	The switch that tells function to execute with key parameters.
     .Parameter KeyFile
     	The path and file name to the key file used for key based
     	authentication.
     .Parameter User
     	The switch that tells function to execute with user parameters.
     .Parameter Password
     	The password string used for the connection.
 	#> 
     [CmdletBinding(DefaultParameterSetName="KeySet")]
     param(
 	[Parameter(Mandatory=$True)][String]$Host,
 	[Parameter(Mandatory=$True)][String]$UserName,
 	[Parameter(ParameterSetName="KeySet")][Switch]$Key,
         [Parameter(ParameterSetName="KeySet", Mandatory=$True)][String]$KeyFile,
         [Parameter(ParameterSetName="User")][Switch]$User,
         [Parameter(ParameterSetName="User", Mandatory=$True)][String]$Password,
 	[Switch]$SCP,
 	[Switch]$PassThru
         )
 	
 	try {
 		if ($SCP){
 			$ConnectType = "Tamir.SharpSsh.scp"
 			}
 		else{
 			$ConnectType = "Tamir.SharpSsh.SshShell"
 			}
 		
 		if ($Key -and (Test-Path $KeyFile)){
 			$Global:SharpSession = New-Object $ConnectType $Host, $UserName
 			$Global:SharpSession.AddIdentityFile((Resolve-Path $KeyFile))
 			}
 		else{
 			$Global:SharpSession = New-Object $ConnectType $Host, $UserName, $Password
 			}
 			
 		$Global:SharpSession.Connect()
 		
 		if ($ConnectType -eq "Tamir.SharpSsh.SshShell"){
 			$Global:SharpSession.RemoveTerminalEmulationCharacters = $True
 			}
 		
 		if($PassThru){
 			return $Global:SharpSession
 			}
 		}
 	catch {
 		throw $Error[0]
 		}
 	}
 
 #-------------------------------------------------
 #-------------------------------------------------
 #-------------------------------------------------
 function Transfer-SharpFile {
     .Synopsis
     	Used to tranfser(get or put) a file using a valid SCP SharpSession.
     .Description
     	Uses SharpSSH to tranfer (get or put) a file via SCP.
     .Parameter From
     	The full path and file name to the file you are transferring.
     .Parameter To
     	The full path and file name to where you want to copy the file to.
 	#> 
     param(
 	[Parameter(Mandatory=$True)][String]$From,
 	[Parameter(Mandatory=$True)][String]$To,
 	[Switch]$Get,
 	[Switch]$Put
 		)
 	
 	try {
 		if (!($Get -or $Put)) {
 			throw Write-Host "Get or Put must be defined!" -ForeGroundColor Red
 			}
 	
 		if ($SharpSession.GetType().Fullname -ne "Tamir.SharpSsh.Scp"){
 			throw Write-Host "Not currently connected using SCP!" -ForeGroundColor Red
 			}
 		else{
 			$Global:SharpSessiontransferDone = $Null
 			$Global:SharpSessiontotalBytes = $Null
 			$Global:SharpSessiontransferredBytes = $Null
 						
 			[void] (Register-ObjectEvent  -InputObject $SharpSession -EventName OnTransferProgress -SourceIdentifier Scp.OnTransferProgress `
 				-Action {$Global:SharpSessiontotalBytes = $args[3]; $Global:SharpSessiontransferredBytes = $args[2]})
 			[void] (Register-ObjectEvent  -InputObject $SharpSession -EventName OnTransferEnd -SourceIdentifier Scp.OnTransferEnd `
 				-Action {$Global:SharpSessiontransferDone = $True})
 	
 			if ($Get){
 				$SharpSession.Get($From, $To)
 				
 				try{
 					do {Write-Progress -Activity "Get - $($From)" -Status "Copying" `
 						-PercentComplete (($SharpSessiontransferredBytes/$SharpSessiontotalBytes)*100)} while ($SharpSessiontransferDone -eq $Null)
 					}
 				catch{
 					Write-Warning "File is not there..."
 					}
 				}
 			else{
 				$SharpSession.Put($From, $To)
 				
 				try{
 					do {Write-Progress -Activity "Get - $($From)" -Status "Copying" `
 						-PercentComplete (($SharpSessiontransferredBytes/$SharpSessiontotalBytes)*100)} while ($SharpSessiontransferDone -eq $Null)
 					}
 				catch{
 					Write-Warning "File is not there..."
 					}
 				}
 			}
 		}
 	catch {
 		throw $Error[0]
 		}
 	finally {
 		if (Get-EventSubscriber -SourceIdentifier Scp.OnTransferProgress){
 			[void] (Unregister-Event -SourceIdentifier Scp.OnTransferProgress)
 			}
 		if (Get-EventSubscriber -SourceIdentifier Scp.OnTransferEnd){
 			[void] (Unregister-Event -SourceIdentifier Scp.OnTransferEnd)
 			}
 		}
 	}
 	
 #-------------------------------------------------
 #-------------------------------------------------
 #-------------------------------------------------
 function Remove-SharpSession {
     .Synopsis
     	Used to close an active SSH or SCP Session.
     .Description
     	Uses SharpSSH to close an active SSH or SCP Session.
 	#> 
 	
 	try {
 		if($SharpSession){
 			if ($SharpSession.GetType().Fullname -eq "Tamir.SharpSsh.SshShell"){
 				$SharpSession.WriteLine("exit")
 				sleep -milli 500
 	
 				if($SharpSession.ShellOpened) {
 					Write-Warning "Shell didn't exit cleanly, closing anyway." 
 					}
 				
 				$SharpSession.Close()
 				}
 			else{
 				$SharpSession.Close()
 				}
 			}
 		else{
 			throw "There is no session to remove!"
 			}
 		}
 	catch {
 		throw $Error[0]
 		}
 	finally {
 		$Global:SharpSession = $Null
 		}
 	}	
 	
 ##################################################
 ##################################################
 Export-ModuleMember New-SharpSession
 Export-ModuleMember Transfer-SharpFile
 Export-ModuleMember Remove-SharpSession
`

