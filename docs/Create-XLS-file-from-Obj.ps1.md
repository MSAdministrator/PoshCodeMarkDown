---
Author: peter kriegel
Publisher: 
Copyright: 
Email: 
Version: 1.0.0
Encoding: utf-8
License: cc0
PoshCode ID: 4369
Published Date: 2015-08-07t10
Archived Date: 2015-12-06t03
---

# create xls file from obj - 

## Description

function to create a tab delimited excel .xls file from objects.

## Comments



## Usage



## TODO



## function

`export-xls`

## Code

`#
 #
 Function Export-XLS {
 <#
 	.SYNOPSIS
 		Function to create a Tab delimited Excel .xls File from Objects.
 		You can use this function just as the Export-CSV cmdlet
 
 	.DESCRIPTION
 		Function to create a Tab delimited Excel .xls File from Objects.
 		You can use this function just as the Export-CSV cmdlet
 		
 		If you store Tab delimited data, that is arranged in rows and columns,
 		in a file with the file extension .xls, Excel Versions from 97 to 2003 open 
 		this file as a Excel workbook.
 		Even the newer Versions of Excel (2007 -2013) support this File Format.
 		Excel (2007 -2013) displays only a warning during open the file.
 		
 		This Funktion creates such a file with the file extension .xls (Excel Version 97 to 2003)
 		
 		Many users believe CSV is just another type of Excel file, however this is not the case.
 		Microsoft Excel will automatically convert data columns into the format that
 		it thinks is best when opening a CSV or a Tab delimited data file.
 		For example, Excel will remove leading Zeros of Numbers, change Data/Time Formats or uses
 		the sientific numberformat for large Numbers and others.
 		This can go unnoticed in large data sets.
 		If you enclose each data field in double quotes and putt an '=' before the double quotes
 		this will force the Excel parser to Import the data as type of Text.
 		Example: ="<data>"
 		So the data will leaved unchanged / unconverted.
 		Excel 2007 and above need a slight different data masking which looks like so: "=""<data>""".
 		This data format also works with Excel 97.
 		
 		If you don�t want that Excel changes / converts the data, you can use the -NoReformat parameter of this function.
 		This will store the Tab delimited file with the the described dataformat "=""<data>""" .
 		So you can open the .xls file without changing Data
 		
 	.PARAMETER  Encoding
 		Specifies the encoding for the exported XLS file.
 		Valid values are: Unicode, UTF7, UTF8, ASCII, UTF32, BigEndianUnicode, Default, and OEM. The default is ASCII.
 
 	.PARAMETER  Force
 		Overwrites the file specified in path without prompting.
 		
 	.PARAMETER  InputObject
 		Specifies the objects to export as Tab delimited strings.
 		Enter a variable that contains the objects or type a command or expression that gets the objects.
 		You can also pipe objects to Export-XLS.
 		
 	.PARAMETER  NoClobber
 		Do not overwrite (replace the contents) of an existing file.
 		By default, if a file exists in the specified path, Export-XLS overwrites the file without warning.
 		
 	.PARAMETER  Path
 		Specifies the path to the XLS output file. This parameter is required.
 		
 	.PARAMETER  NoReformat
 		This will store the XLS Tab delimited output file, with the the described dataformat "=""<data>""" .
 		So you can open the XLS file without that Excel reformats the data
 
 	.EXAMPLE
 		PS C:\> Get-Process | Export-XLS -Path 'D:\Temp\test.xls' -NoReformat
 		
 		Create a Tab delimited XLS File. Because the NoReformat parameter given,
 		Excel do not reformat the data fields during the open the file.
 
 	.EXAMPLE
 		PS C:\> Import-Csv -Path 'D:\temp\Processes.csv' -Delimiter ';' | Export-XLS -Path 'D:\Temp\test.xls'
 		
 		Converts a CSV file into a Tab delimited XLS File. Because the NoReformat parameter is not given.
 		Excel is free to reformat the data fields during the open the file.
 
 	.INPUTS
 		The input type is the type of the objects that you can pipe to the cmdlet.
 		System.Management.Automation.PSObject
 		You can pipe any object with an Extended Type System (ETS) adapter to Export-XLS.
 
 	.OUTPUTS
 		The output type is the type of the objects that the cmdlet emits.
 		System.String
 		The XLS list is sent to the file designated in the Path parameter.
 
 	.NOTES
 		Author: Peter Kriegel
 		Version: 1.0.0 07.August.2013
 
 #>
 
 [CmdletBinding(DefaultParameterSetName='Delimiter', SupportsShouldProcess=$true, ConfirmImpact='Medium')]
 param(
     [Parameter(Mandatory=$true, ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true)]
     [System.Management.Automation.PSObject]
     ${InputObject},
 
     [Parameter(Mandatory=$true, Position=0)]
     [Alias('PSPath','FilePath')]
     [System.String]
     ${Path},
 
     [Switch]
     ${NoClobber},
 
     [ValidateSet('Unicode','UTF7','UTF8','ASCII','UTF32','BigEndianUnicode','Default','OEM')]
     [System.String]
     ${Encoding} = 'ASCII',
 	
 	[Switch]
     ${Force},
 	
 	[Switch]
 	${NoReformat}
 
 	)
 
 	begin {
 		
 		If(-not (Test-Path $Path -IsValid)) {
 			Write-Error "Path not valid: $Path"
 			return
 		}
 		
 		$Path = [System.IO.Path]::ChangeExtension($Path,'xls')
 		
 		
 		$PSBoundParameters.Add('NoTypeInformation',$NoTypeInformation)
 		$PSBoundParameters.Add('Delimiter',[System.Char]"`t")
 		
 		$HeaderWritten = $False
 			
 		If(-not ($NoReformat.IsPresent)) {
 			try {
 		        $outBuffer = $null
 		        if ($PSBoundParameters.TryGetValue('OutBuffer', [ref]$outBuffer))
 		        {
 		            $PSBoundParameters['OutBuffer'] = 1
 		        }
 		        $wrappedCmd = $ExecutionContext.InvokeCommand.GetCommand('Export-Csv', [System.Management.Automation.CommandTypes]::Cmdlet)
 		        $scriptCmd = {& $wrappedCmd @PSBoundParameters }
 		        $steppablePipeline = $scriptCmd.GetSteppablePipeline($myInvocation.CommandOrigin)
 		        $steppablePipeline.Begin($PSCmdlet)
 				$HeaderWritten = $True
 		    } catch {
 		        throw
 		    }
 		} Else {
 			Try {
 				Out-File -FilePath $Path -Encoding $Encoding -NoClobber:($NoClobber.IsPresent) -Force:($Force.IsPresent) -ErrorAction Stop
 			} Catch {
 				$_
 				Return
 			}
 		}
 	}
 
 	process	{
 	    
 		If(-not ($NoReformat.IsPresent)) {
 			try {
 		        $steppablePipeline.Process($_)
 		    } catch {
 		        throw
 		    }
 		} Else {
 			
 			$StringBuilder = New-Object System.Text.StringBuilder -ArgumentList ""
 		
 			If($InputObject.GetType().Name -match 'byte|short|int32|long|sbyte|ushort|uint32|ulong|float|double|decimal|string') {
 				Add-Member -InputObject $InputObject -MemberType NoteProperty -Name ($InputObject.GetType().Name) -Value $InputObject
 			}
 					
 			If(-not $HeaderWritten) {
 				$InputObject | ConvertTo-Csv -NoTypeInformation -Delimiter "`t" | Select-Object -First 1 | Out-File -FilePath $Path -Encoding $Encoding -Append
 	        	$HeaderWritten = $True
 	    	}
 			
 			$Null = $StringBuilder.Remove(0,$StringBuilder.Length)
 	    
 	    	ForEach($Prop in $InputObject.psobject.Properties) {
 	            $Null = $StringBuilder.Append([String]('"=""{0}"""{1}' -f  ([String]$Prop.Value),"`t"))
 	    	}
 	    
 	    	If($StringBuilder.Length -gt 0 ) {          
 	        	$Null = $StringBuilder.Remove($StringBuilder.Length -1 ,1)
 	        	$StringBuilder.ToString() | Out-File -FilePath $Path -Append -Encoding $Encoding
 	    	}
 		}
 	}
 
 	end	{
 	    If(-not ($NoReformat.IsPresent)) {
 			try {
 		        $steppablePipeline.End()
 		    } catch {
 		        throw
 		    }
 		}	
 	}
  
 }
`

