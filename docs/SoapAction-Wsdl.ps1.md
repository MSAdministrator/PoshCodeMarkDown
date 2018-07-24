---
Author: daniel cheng
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5157
Published Date: 2014-05-09t20
Archived Date: 2014-05-15t01
---

# soapaction wsdl - 

## Description

obtain the correct soapaction string when a webservice method is passed (read from a soap request).

## Comments



## Usage



## TODO



## function

`find-soapaction`

## Code

`#
 #
 function Find-SoapAction
 {
     [CmdletBinding(PositionalBinding=$false)]
     param(
         [string]$method,
     )
     
  
     Add-Type -AssemblyName "System.Web.Services"
     $webClient = New-Object -TypeName System.Net.WebClient
     $serviceDescriptionCollection = New-Object -TypeName System.Web.Services.Description.serviceDescriptionCollection
 
     function private:Get-ServiceDescription([uri]$WsdlLocation)
     {
         $error.Clear()
         Write-Verbose "Trying $($WsdlLocation).."
         if ($WsdlLocation.Scheme -eq "https"){[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}}
         try {
             $wsdlStream = $WebClient.OpenRead($WsdlLocation)
             $serviceDescription = [Web.Services.Description.ServiceDescription]::Read($wsdlStream)
             $wsdlStream.Close()
         } catch {
             Write-Verbose "$($WsdlLocation) retrieval failed!"
             return $false
         }
         Write-Verbose "$($WsdlLocation) retrieval success!"
         return $serviceDescription
     }
 
     function private:Get-BindingOperation
     {
         param($serviceDescription)
         foreach ($message in $serviceDescription.Messages)
         {
             Write-Verbose "comparing $($message.parts.element.name) with $($method)"
             if ($message.parts.element.name -eq $method)
             {
                 foreach ($PortTypeOperation in $serviceDescription.PortTypes.Operations.Messages) {
                     Write-Verbose "comparing $($PortTypeOperation.Message.Name) with $($message.name)"
                     if ($PortTypeOperation.Message.Name -eq $message.Name)
                     {
                         Write-Verbose $PortTypeOperation.operation.name
                         return $PortTypeOperation.operation.name
                     }
                 }
             }
         }
     }
 
     function private:Find-BindingOperationMapping
     {
         param($bindingOperation, $serviceDescription)
         foreach ($servicePortBinding in $serviceDescription.services.ports.Binding)
         {
             foreach ($Binding in $serviceDescription.Bindings)
             {
                 Write-Verbose "comparing $($servicePortBinding.Name) with $($Binding.Name)"
                 if (($servicePortBinding.Name -eq $Binding.Name) -and ($servicePortBinding.name -notlike "*soap12*"))
                 {
                     foreach ($Operation in $Binding.Operations)
                     {
                         if ($Operation.Name -eq $bindingOperation)
                         {
                             return ([uri]$Operation.Extensions.soapAction).AbsoluteUri
                         }
                     }
                 }
     
             }
         }
     }
 
     if ($wsdl.IsFile)
     {
         $location = Get-ServiceDescription -WsdlLocation $wsdl
     } else {
         try {
             $location = Get-ServiceDescription -WsdlLocation ($wsdl.AbsoluteUri -replace "\?wsdl", "?singleWsdl")
             if (![bool]$location) {throw "error"}
         } catch {
             $location = Get-ServiceDescription -WsdlLocation $wsdl
         }
     }    
     
     if ($location.GetType().BaseType -eq [System.Web.Services.Description.NamedItem])
     {
         try {
             [void]$serviceDescriptionCollection.add($location)
         } catch {
             Write-Verbose "[serviceDescriptionCollection] $($error)"
         } finally {
             if ([bool]$location.imports.count)
             {
                 foreach ($importLocation in $location.Imports.location)
                 {
                     Write-Verbose "Importing from $($importLocation).."
                     try {
                         [void]$serviceDescriptionCollection.add((Get-ServiceDescription $importLocation))
                     } catch {
                         Write-Verbose "[importLocation] $($error)"
                     }
                 }
             }
         }
     }
 
     foreach ($serviceDescription in $serviceDescriptionCollection)
     {
         $bindingOperation = Get-BindingOperation -serviceDescription $serviceDescription
         if ($bindingOperation)
         {
             foreach ($serviceDescription in $serviceDescriptionCollection)
             {
                 Find-BindingOperationMapping -bindingOperation $bindingOperation -serviceDescription $serviceDescription
             }
             
         }
     }
 }
`

