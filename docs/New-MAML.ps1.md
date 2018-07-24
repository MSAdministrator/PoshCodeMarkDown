---
Author: chad miller
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4212
Published Date: 2013-06-21t07
Archived Date: 2013-06-25t03
---

# new-maml - 

## Description

generates external maml powershell help file for any loaded cmdlet or function

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 #
 
 
 param ($commandName)
 $scriptRoot = Split-Path (Resolve-Path $myInvocation.MyCommand.Path)
 . $scriptRoot\New-XML.ps1
 
 [XNamespace]$helpItems="http://msh"
 [XNamespace]$maml="http://schemas.microsoft.com/maml/2004/10"
 [XNamespace]$command="http://schemas.microsoft.com/maml/dev/command/2004/10"
 [XNamespace]$dev="http://schemas.microsoft.com/maml/dev/2004/10"
 $parameters =  get-command $commandName | %{$commandName=$_.Name;$_.parameters} | %{$_.Values}
 
 New-Xml helpItems -schema "maml" {
     xe ($command + "command") -maml $maml -command $command -dev $dev {
             xe ($command + "details") {
                 xe ($command + "name") {"$commandName"}
                 xe ($maml + "description") {
                     xe ($maml + "para") {"TODO Add Short description"}
                 }
                 xe ($maml + "copyright") {
                     xe ($maml + "para") {}
                 }
                 xe ($command + "verb") {"$(($CommandName -split '-')[0])"}
                 xe ($command + "noun") {"$(($commandName -split '-')[1])"}
                 xe ($dev + "version") {}
             }
             xe ($maml + "description") {
                 xe ($maml + "para") {"TODO Add Long description"}
             }
             xe ($command + "syntax") {
                 xe ($command + "syntaxItem") {
                 $parameters | foreach { 
                     xe ($command + "name") {"$commandName"}
                         xe ($command + "parameter") -require "false" -variableLength "false" -globbing "false" -pipelineInput "false" -postion "0" {
                             xe ($maml + "name") {"$($_.Name)"}
                             xe ($maml + "description") {
                                 xe ($maml + "para") {"TODO Add $($_.Name) Description"}
                             }
                             xe ($command + "parameterValue") -required "false" -variableLength "false" {"$($_.ParameterType.Name)"}
                         }
                     }
                 }
             }
             xe ($command + "parameters") {
                 $parameters | foreach { 
                 xe ($command + "parameter") -required "false" -variableLength "false" -globbing "false" -pipelineInput "false (ByValue)" -position "0" {
                     xe ($maml + "name") {"$($_.Name)"}
 		    xe ($maml + "description") {
 			xe ($maml + "para") {"TODO Add $($_.Name) Description"}
                     }
 		    xe ($command + "parameterValue") -required "true" -variableLength "false" {"$($_.ParameterType.Name)"}
                     xe ($dev + "type") {
                         xe ($maml + "name") {"$($_.ParameterType.Name)"}
 			xe ($maml + "uri"){}
                     }
 		    xe ($dev + "defaultValue") {}
                 }
                 }
             }
 	    xe ($command + "inputTypes") {
                 xe ($command + "inputType") {
                     xe ($dev + "type") {
                         xe ($maml + "name") {"TODO Add $commandName inputType"}
                         xe ($maml + "uri") {}
                         xe ($maml + "description") {
                             xe ($maml + "para") {}
                         }
                     }
 			xe ($maml + "description") {}
                 }
             }
 	    xe ($command + "returnValues") {
 		xe ($command + "returnValue") {
 		    xe ($dev + "type") {
 		        xe ($maml + "name") {"TODO Add $commandName returnType"}
                         xe ($maml + "uri") {}
                         xe ($maml + "description") {
                             xe ($maml + "para") {}
                         }
                     }
 		    xe ($maml + "description") {}
 		}
 	    }
             xe ($command + "terminatingErrors") {}
 	    xe ($command + "nonTerminatingErrors") {}
 	    xe ($maml + "alertSet") {
 		xe ($maml + "title") {}
 		xe ($maml + "alert") {
 		    xe ($maml + "para") {}
                 }
             }
             xe ($command + "examples") {
 		xe ($command + "example") {
                     xe ($maml + "title") {"--------------  EXAMPLE 1 --------------"}
                     xe ($maml + "introduction") {
                         xe ($maml + "para") {"C:\PS&gt;"}
                     }
                     xe ($dev + "code") {"TODO Add $commandName Example code"}
                     xe ($dev + "remarks") {
                         xe ($maml + "para") {"TODO Add $commandName Example Comment"}
                         xe ($maml + "para") {}
                         xe ($maml + "para") {}
                         xe ($maml + "para") {}
                         xe ($maml + "para") {}
                     }
                     xe ($command + "commandLines") {
                         xe ($command + "commandLine") {
                             xe ($command + "commandText") {}
                         }
                     }
                 }
             }   
             xe ($maml + "relatedLinks") {
                 xe ($maml + "navigationLink") {
 		    xe ($maml + "linkText") {"$commandName"}
 		    xe ($maml + "uri") {}
                 }
             }
         }
     }
`

