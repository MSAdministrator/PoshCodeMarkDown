---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.4
Encoding: ascii
License: cc0
PoshCode ID: 3029
Published Date: 2012-10-27t21
Archived Date: 2012-12-23t23
---

# scripttransforms module - 

## Description

powershell module that allows scripters to define argument transformation attributes in simple powershell syntax.  this is an extension developed by beefarino of my scripttransform example at http

## Comments



## Usage



## TODO



## function

`new-parametertransform`

## Code

`#
 #
 function New-ParameterTransform {
 #.Synopsis
 #.Description
 #.Example
 #
 #
 #
 #
 #
 #
 #.Notes
 #
 	[cmdletbinding()]
 	param(
 		[Parameter(Mandatory=$true,Position=0)]
 		[string] $name,
 		[Parameter(Mandatory=$true, Position=1)]
 		[scriptblock] $script
 	)
 
 end {
 	$className = [Convert]::ToBase64String( ([Guid]::NewGuid().ToByteArray()) ).Replace('+',([char]241)).Replace('/',([char]223)).TrimEnd('=')
 
 	$Type = Add-Type -Passthru -TypeDefinition @"
 using System;
 using System.ComponentModel;
 using System.Management.Automation;
 using System.Collections.ObjectModel;
 
 [AttributeUsage(AttributeTargets.Field | AttributeTargets.Property)]
 public class ${name}Attribute${className} : ArgumentTransformationAttribute {
    private ScriptBlock _scriptblock;
    private string _noOutputMessage = "Transform Script had no output.";
 
    public ${name}Attribute${className}() {
       this.Script = ScriptBlock.Create( @"
 	  	$($script.ToString() -replace '"','""' )
 " );
    }
    public override string ToString() {
       return Script.ToString();
    }
    
    public override Object Transform( EngineIntrinsics engine, Object inputData) {
       Collection<PSObject> output;
       try {
          output = engine.InvokeCommand.InvokeScript( engine.SessionState, Script, inputData );
       } catch (ArgumentTransformationMetadataException) {
          throw;
       } catch (Exception e) {
          throw new ArgumentTransformationMetadataException(
             string.Format("Transform Script threw an exception ('{0}'). See `$Error[0].Exception.InnerException.InnerException for more details.", e.Message),
             e);
       }
       if(output == null || output.Count == 0 ) { 
          throw new ArgumentTransformationMetadataException(NoOutputMessage);
       }
       
       if(output.Count > 1) 
       {
          Object[] transformed = new Object[output.Count];
          for(int i =0; i < output.Count;i++) {
             if(output[i] == null) {
                transformed[i] = null;
             } else {
                transformed[i] = output[i].BaseObject;
             }
          }
          return transformed;
       }
       else // (output.Count == 1) 
       {
          if(output[0] == null) {
             return null;
          }
          return output[0].BaseObject;
       }
    }
    
    public ScriptBlock Script {
       get { return _scriptblock; }
       set { _scriptblock = value; }
    }
    
    public string NoOutputMessage {
       get { return _noOutputMessage; }
       set { _noOutputMessage = value; }
    }  
 }
 "@
 
 	$xlr8r = [psobject].assembly.gettype("System.Management.Automation.TypeAccelerators")
 	if($xlr8r::AddReplace) { 
 		$xlr8r::AddReplace( ${name}, $Type) 
 	} else {
 		$null = $xlr8r::Remove( ${name} )
 		$xlr8r::Add( ${name}, $Type)
 	}
 }}
 
 New-Alias -Name Transform -Value New-ParameterTransform;
 Export-ModuleMember -Alias Transform -Function *
`

