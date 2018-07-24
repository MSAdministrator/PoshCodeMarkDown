---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1057
Published Date: 2010-04-23t19
Archived Date: 2016-03-05t14
---

# start-processasadministr - 

## Description

original author

## Comments



## Usage



## TODO



## function

`start-processasadministrator`

## Code

`#
 #
 function Start-ProcessAsAdministrator
 {
     <#
     .ForwardHelpTargetName Start-Process
     .ForwardHelpCategory Cmdlet
     #>
     [CmdletBinding(DefaultParameterSetName='Default')]
     param(
     [Parameter(Mandatory=$true, Position=0)]
     [Alias('PSPath')]
     [ValidateNotNullOrEmpty()]
     [System.String]
     $FilePath,
 
     [Parameter(Position=1)]
     [Alias('Args')]
     [ValidateNotNullOrEmpty()]
     [System.String[]]
     $ArgumentList,
 
     [Parameter(ParameterSetName='Default')]
     [Alias('RunAs')]
     [ValidateNotNullOrEmpty()]
     [System.Management.Automation.PSCredential]
     $Credential,
 
     [ValidateNotNullOrEmpty()]
     [System.String]
     $WorkingDirectory,
 
     [Parameter(ParameterSetName='Default')]
     [Alias('Lup')]
     [Switch]
     $LoadUserProfile,
 
     [Parameter(ParameterSetName='Default')]
     [Alias('nnw')]
     [Switch]
     $NoNewWindow,
 
     [Switch]
     $PassThru,
 
     [Parameter(ParameterSetName='Default')]
     [Alias('RSE')]
     [ValidateNotNullOrEmpty()]
     [System.String]
     $RedirectStandardError,
 
     [Parameter(ParameterSetName='Default')]
     [Alias('RSI')]
     [ValidateNotNullOrEmpty()]
     [System.String]
     $RedirectStandardInput,
 
     [Parameter(ParameterSetName='Default')]
     [Alias('RSO')]
     [ValidateNotNullOrEmpty()]
     [System.String]
     $RedirectStandardOutput,
 
     [Switch]
     $Wait,
 
     [Parameter(ParameterSetName='UseShellExecute')]
     [ValidateNotNullOrEmpty()]
     [System.Diagnostics.ProcessWindowStyle]
     $WindowStyle,
 
     [Parameter(ParameterSetName='Default')]
     [Switch]
     $UseNewEnvironment   
     )    
     
     process {
         $psBoundParameters = $psBoundParameters += @{"Verb"="runas"}    
         Start-Process @psBoundParameters
     }
 }
`

