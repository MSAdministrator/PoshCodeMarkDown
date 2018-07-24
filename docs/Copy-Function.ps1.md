---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1410
Published Date: 
Archived Date: 2009-10-26t15
---

# copy-function - 

## Description

copy a function from your current session to another session. allows you to easily copy function definitions into remote pssessions.

## Comments



## Usage



## TODO



## function

`copy-function`

## Code

`#
 #
 
 <#
    .Synopsis
       Copy a function from the current session to another session
    .Description
       Copies a function deffinition from the current session into any other session
    .Parameter Session
       The session(s) you want to define the function in
    .Parameter Name
       The Name of the function to copy
    .Parameter Definition
       The optional definition of the function. This is used to allow copying via the pipeline.
    .Parameter Force
       Overwrite existing functions in the session.
    .Parameter Passthru
       Output the FunctionInfo from the session
    .Example
       Copy-Function -Session $Session1 -Name Prompt -Force
         
       Copies the prompt function from the current session into the specified session, overwriting the existing prompt function.
    .Example
       Get-Command -Type Function | Copy-Function $Session1
       
       Copies all of the functions from the current session into the new session.
    .Notes
       
     
 #>
 function Copy-Function {
 [CmdletBinding(SupportsShouldProcess=$true)]
 Param(
    [Parameter(Mandatory=$true, Position=0)]
    [System.Management.Automation.Runspaces.PSSession[]]
    $Session, 
 
    [Parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Mandatory=$true,Position=1)]
    [String]
    $name, 
 
    [Parameter(ValueFromPipelineByPropertyName=$true,Mandatory=$false)]
    [String]
    $Definition,
    
    [Switch]
    $Force,
    
    [Switch]
    $Passthru
    
 )
 
 Process { 
    if( $PSCmdlet.ShouldProcess("Copied function $Name to sessions: $(($Session|select -expand Name) -join ', ')","Copy function `"$Name`"?","Copying functions to sessions: $(($Session|select -expand Name) -join ', ')") ) {
       if(!$Definition){ $Definition = (gcm -type function $name).Definition }
       If(!$Passthru) {
          Invoke-Command { 
             Param($name, $value, $force) 
             $null = new-item function:"$name" -value $value -force:$force
          } -Session $Session -ArgumentList $Name,$Definition,$Force
       } else {
          Invoke-Command { 
             Param($name, $value, $force) 
             new-item function:"$name" -value $value -force:$force
          } -Session $Session -ArgumentList $Name,$Definition,$Force
       }   
    }
 }
 }
`

