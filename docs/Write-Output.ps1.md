---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: ms-pl, ms-rl, gpl 2, or bsd
PoshCode ID: 2300
Published Date: 
Archived Date: 2010-10-20t09
---

# write-output - 

## Description

a write-output replacement with the option to output collections that don’t unwrap.

## Comments



## Usage



## TODO



## function

`write-output`

## Code

`#
 #
 ########################################################################
 
 function Write-Output {
 [CmdletBinding()]
 param(
     [Parameter(Mandatory=$true, Position=0, ValueFromPipeline=$true)]
     [AllowEmptyCollection()]
     [AllowNull()]
     [System.Management.Automation.PSObject]
     ${InputObject}
 ,
 	[Parameter()]
 	[Switch]
 	[Alias("PreventUnrolling")]
 	$AsCollection
 )
 
 begin
 {
 	$Collection = New-Object System.Collections.Generic.List[PSObject]
 }
 
 process
 {
 	if($InputObject) {
 		if($AsCollection) {
 			$Collection.AddRange( ([PSObject[]]@($InputObject)) )
 		} else {
 			$PSCmdlet.WriteObject( $InputObject, $true )
 		}
 	}
 }
 
 end
 {
 	if($AsCollection) {
 		$PSCmdlet.WriteObject( $Collection, $false )
 	}
 }
 <#
 .SYNOPSIS
     Sends the specified objects to the next command in the pipeline. If the command is the last 
     command in the pipeline, the objects are displayed in the console.
 .DESCRIPTION
     The Write-Output function sends the specified object down the pipeline to the next command. If the command is the last command in the pipeline, the object is displayed in the console.
     
     Write-Output sends objects down the primary pipeline, also known as the "output stream" or the "success pipeline." To send error objects down the error pipeline, use Write-Error.
     
 	This function provides a pair of enhancements to the built-in Write-Output. First, it offers the option of Passthru, which causes collections to be output without unrolling (that is, output as collections to the pipeline), and second, it includes an option to collect input from the pipeline and output it all at once at the end.
 .PARAMETER InputObject
     Specifies the objects to send down the pipeline. Enter a variable that contains the objects, or type a command or expression that gets the objects.
 .PARAMETER AsCollection
 	Specifies that pipeline input should be collected and output all at once at the end. 
 	This effectively allows you to take streaming input and turn it into a collection, or output an array from your function without unrolling.
 	
 	Note that this causes the output to be a List[PSObject].
 .EXAMPLE    
     $p = get-process
     
     C:\PS>write-output $p
     
     Description
     -----------
     These commands get objects representing the processes running on the computer and display the objects on the console.
     
 .EXAMPLE    
     $p = get-process | Select-Object -First 10
     
     C:\PS>write-output $p  | ForEach-Object { $_.GetType() }
 		
 	IsPublic IsSerial Name       BaseType                             
 	-------- -------- ----       --------                             
 	True     False    Process    System.ComponentModel.Component      
 	True     False    Process    System.ComponentModel.Component      
 	True     False    Process    System.ComponentModel.Component      
 	True     False    Process    System.ComponentModel.Component      
 	True     False    Process    System.ComponentModel.Component      
 	True     False    Process    System.ComponentModel.Component      
 	True     False    Process    System.ComponentModel.Component      
 	True     False    Process    System.ComponentModel.Component      
 	True     False    Process    System.ComponentModel.Component      
 	True     False    Process    System.ComponentModel.Component
 
     C:\PS>write-output $p -AsCollection | ForEach-Object { $_.GetType() }
     
 	
 	IsPublic IsSerial Name       BaseType                             
 	-------- -------- ----       --------                             
 	True     True     List`1     System.Array    
 	
     Description
     -----------
     These commands demonstrate the difference between using and not using -AsCollection. Without it, each object passes through the ForEach and it's type is output. With -AsCollection, only a single object is output, with a collection type of List`1
     
 .EXAMPLE
     Get-Process | Select-Object -First 10 | Write-Output -AsCollection | ForEach-Object { $_.GetType() }
     
     Description
     -----------
     This command pipes the first ten processes to the ForEach-Object, demonstrating that -AsCollection will force the output to be a collection.
     
 .EXAMPLE
     Get-Process | Write-Output -AsCollection | ForEach-Object { $_.GetType() }
     
     Description
     -----------
     This command collects all of the processes before outputting them to the ForEach-Object as a List[PSObject]
 #>
 }
`

