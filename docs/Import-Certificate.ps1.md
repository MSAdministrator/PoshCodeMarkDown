---
Author: anti121
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5894
Published Date: 2016-06-15t14
Archived Date: 2016-05-19t04
---

# import-certificate - 

## Description

function to import security certificates.

## Comments



## Usage



## TODO



## function

`import-certificate`

## Code

`#
 #
 
 function Import-Certificate
 {
 	param
 	(
 		[IO.FileInfo] $CertFile = $(throw "Paramerter -CertFile [System.IO.FileInfo] is required."),
 		[string[]] $StoreNames = $(throw "Paramerter -StoreNames [System.String] is required."),
 		[switch] $LocalMachine,
 		[switch] $CurrentUser,
 		[string] $CertPassword,
 		[switch] $Verbose
 	)
 	
 	begin
 	{
 		[void][System.Reflection.Assembly]::LoadWithPartialName("System.Security")
 	}
 	
 	process 
 	{
         if ($Verbose)
 		{
             $VerbosePreference = 'Continue'
         }
     
 		if (-not $LocalMachine -and -not $CurrentUser)
 		{
 			Write-Warning "One or both of the following parameters are required: '-LocalMachine' '-CurrentUser'. Skipping certificate '$CertFile'."
 		}
 
 		try
 		{
 			if ($_)
             {
  @@               $certfile = $_.FullName
             }
             $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2 $certfile,$CertPassword
 		}
 		catch
 		{
 			Write-Error ("Error importing '$certfile': $_ .") -ErrorAction:Continue
 		}
 			
 		if ($cert -and $LocalMachine)
 		{
 			$StoreScope = "LocalMachine"
 			$StoreNames | ForEach-Object {
 				$StoreName = $_
 				if (Test-Path "cert:\$StoreScope\$StoreName")
 				{
 					try
 					{
 						$store = New-Object System.Security.Cryptography.X509Certificates.X509Store $StoreName, $StoreScope
 						$store.Open([System.Security.Cryptography.X509Certificates.OpenFlags]::ReadWrite)
 						$store.Add($cert)
 						$store.Close()
 						Write-Verbose "Successfully added '$certfile' to 'cert:\$StoreScope\$StoreName'."
 					}
 					catch
 					{
 						Write-Error ("Error adding '$certfile' to 'cert:\$StoreScope\$StoreName': $_ .") -ErrorAction:Continue
 					}
 				}
 				else
 				{
 					Write-Warning "Certificate store '$StoreName' does not exist. Skipping..."
 				}
 			}
 		}
 		
 		if ($cert -and $CurrentUser)
 		{
 			$StoreScope = "CurrentUser"
 			$StoreNames | ForEach-Object {
 				$StoreName = $_
 				if (Test-Path "cert:\$StoreScope\$StoreName")
 				{
 					try
 					{
 						$store = New-Object System.Security.Cryptography.X509Certificates.X509Store $StoreName, $StoreScope
 						$store.Open([System.Security.Cryptography.X509Certificates.OpenFlags]::ReadWrite)
 						$store.Add($cert)
 						$store.Close()
 						Write-Verbose "Successfully added '$certfile' to 'cert:\$StoreScope\$StoreName'."
 					}
 					catch
 					{
 						Write-Error ("Error adding '$certfile' to 'cert:\$StoreScope\$StoreName': $_ .") -ErrorAction:Continue
 					}
 				}
 				else
 				{
 					Write-Warning "Certificate store '$StoreName' does not exist. Skipping..."
 				}
 			}
 		}
 	}
 	
 	end
 	{ }
 }
`

