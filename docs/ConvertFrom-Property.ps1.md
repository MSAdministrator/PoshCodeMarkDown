---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 3.7
Encoding: ascii
License: cc0
PoshCode ID: 4080
Published Date: 2015-04-07t00
Archived Date: 2015-05-06t14
---

# convertfrom-property - 

## Description

convertfrom-propertystring 3.7 can convert ini files, property files, and other flat key-value data strings into psobjects.

## Comments



## Usage



## TODO



## script

`convertfrom-propertystring`

## Code

`#
 #
 <#
 .SYNOPSIS
    Converts data from flat or single-level property files into PSObjects
 .DESCRIPTION
    Converts delimited string data such as .ini files, or the format-list output of PowerShell, into objects
 .EXAMPLE
    netsh http show sslcert | join-string "`n" | 
    ConvertFrom-PropertyString -ValueSeparator " +: " -AutomaticRecords |
       Format-Table Application*, IP*, Certificate*
             
    Converts the output of netsh show into pseudo-objects, and then uses Format-Table to display them
 .EXAMPLE
    ConvertFrom-PropertyString config.ini
    
    Reads in an ini file (which has key=value pairs), using the default settings
 
    .EXAMPLE
    @"
    ID:3468
    Type:Developer
    StartDate:1998-02-01
    Code:SWENG3
    Name:Baraka
 
    ID:11234
    Type:Management
    StartDate:2005-05-21
    Code:MGR1
    Name:Jax
    "@ |ConvertFrom-PropertyString -sep ":" -RecordSeparator "\r\n\s*\r\n" | Format-Table
 
 
    Code             StartDate       Name            ID              Type           
    ----             ---------       ----            --              ----           
    SWENG3           1998-02-01      Baraka          3468            Developer      
    MGR1             2005-05-21      Jax             11234           Management     
       
    Reads records from a key:value string with records separated by blank lines.
    NOTE that in this example you could also have used -AutomaticRecords or -Count 5 instead of specifying a RecordSeparator
 .EXAMPLE
    @"
    Name=Fred
    Address=Street1
    Number=123
    Name=Janet
    Address=Street2
    Number=345 
    "@ | ConvertFrom-PropertyString -RecordSeparator "`n(?=Name=)"
 
    Reads records from a key=value string and uses a look-ahead record separator to start a new record whenever "Name=" is encountered
    
    NOTE that in this example you could have used -AutomaticRecords or -Count 3 instead of specifying a RecordSeparator 
 .EXAMPLE
    ConvertFrom-PropertyString data.txt -ValueSeparator ":"
    
    Reads in a property file which has key:value pairs
 .EXAMPLE
    Get-Content data.txt -RecordSeparator "`r`n`r`n" | ConvertFrom-PropertyString -ValueSeparator ";"
    
    Reads in a property file with key;value pairs, and records separated by blank lines, and converts it to objects
 .EXAMPLE
    ls *.data | ConvertFrom-PropertyString
    
    Reads in a set of *.data files which have an object per file defined with key:value pairs of properties, one-per line.
 .EXAMPLE
    ConvertFrom-PropertyString data.txt -RecordSeparator "^;(.*?)\r*\n" -ValueSeparator ";"
    
    Reads in a property file with key:value pairs, and sections with a header that starts with the comment character ';'
    
 .NOTES
    3.6   2013 Apr 6
          - Add AllowEmptyValue parameter to supports data that just has a header and a list of properties (without values)
    3.5   2012 July 26
          - Changed defaults so that lines which don't have a -ValueSeparator in them don't get output
          - Changed pipelining so that it works more the way I expect it to, nowadays
          - Fixed some problems with -RecordSeparator getting truncated to a single character when you use a capture group and add it as a PSName property
          Clearly I need to write some test cases around this to make sure that I'm not breaking functionality, because these changes felt like things that should have already worked...
    3.0   2010 Aug 4 
          - Renamed most of the parameters because I couldn't tell which did what from the Syntax help
          - Added a -AutomaticRecords switch which creates new output objects whenevr it encounters a duplicated property
          - Added a -SimpleOutput swicth which prevents the output of the PSChildName property
          - Added a -CountOfPropertiesPerRecord parameter which allows splitting input by count instead of regex or automatic
    2.0   2010 July 9 http://poshcode.org/get/1956
          - changes the output so that if there are multiple instances of the same key, we collect the values in an array
    1.0   2010 June 15 http://poshcode.org/get/1915
          - Initial release
 #>
 [CmdletBinding(DefaultParameterSetName="Data")]
 param(
    [Parameter(Position=99, Mandatory=$true, ValueFromPipeline=$true, ParameterSetName="Data")]
    [Alias("Data","Content","IO")]
    [AllowEmptyString()]
    [string]$InputObject,
 
    [Parameter(Position=0, Mandatory=$true, ValueFromPipelineByPropertyName=$true, ParameterSetName="File")]
    [Alias("PSPath")]
    [string]$FilePath,
 
    [Parameter(ValueFromPipeline=$false, ValueFromPipelineByPropertyName=$false)]
    [Alias("VS","Separator")]
    [String]$ValueSeparator="\s*(?:=|:)\s*",
 
    [Parameter(ValueFromPipeline=$false, ValueFromPipelineByPropertyName=$false)]
    [Alias("PS","Delimiter")]
    [String]$PropertySeparator='(?:\s*\n+\s*)+',
 
    
    [Parameter(ValueFromPipeline=$false, ValueFromPipelineByPropertyName=$false)]
    [Alias("RS")]
    [String]$RecordSeparator='(?:\n|^)\[([^\]]+)\]\s*\n',
    
    [Parameter(ParameterSetName="Data")]
    [Alias("MultiRecords","MR","MultipleRecords","AR","AutoRecords")]
    [Switch]$AutomaticRecords,
    
    [Parameter()]
    [int]$CountOfPropertiesPerRecord,
    
    [Parameter()]
    [Switch]$SimpleOutput,
    
    [Parameter()]
    [Switch]$HasHeader,
    
    [Parameter()]
    [Switch]$HasFooter,
 
    [switch]$AllowEmptyValue
 )
 begin {
    function new-output {
       [CmdletBinding()]
       param(
          [Switch]$SimpleOutput
       ,
          [AllowNull()][AllowEmptyString()]
          [String]$Key
       ,
          [AllowNull()][AllowEmptyString()]
          $FilePath
       )
       end {
          if(!$SimpleOutput -and ("" -ne $Key))  { @{"PSName"=$key} }
          elseif(!$SimpleOutput -and $FilePath)  { @{"PSName"=((get-item $FilePath).PSChildName)} }
          else                                   { @{} }
       }
    }
 
    function out-output {
       [CmdletBinding()]
       param([Hashtable]$output)
       end {
          foreach($k in $Output.Keys | Where { @($Output.$_).Count -eq 1 } ) {
             $Output.$k = @($Output.$k)[0]
          }
          if($output.Count) {
             New-Object PSObject -Property $output
          }
       }
    }
    $OutputCount = 0
    [String]$InputString = [String]::Empty
 
    Write-Verbose "Setting up the regular expressions: `n`tRecord:   '$RecordSeparator'  `n`tProperty: '$PropertySeparator'  `n`tValue:    '$ValueSeparator'"
    [Regex]$ReRecordSeparator   = New-Object Regex ([System.String]"\s*$RecordSeparator\s*"),   ([System.Text.RegularExpressions.RegexOptions]"Multiline,IgnoreCase,Compiled")
    [Regex]$RePropertySeparator = New-Object Regex ([System.String]"\s*$PropertySeparator\s*"), ([System.Text.RegularExpressions.RegexOptions]"Multiline,IgnoreCase,Compiled")
    [Regex]$ReValueSeparator    = New-Object Regex ([System.String]"\s*$ValueSeparator\s*"),    ([System.Text.RegularExpressions.RegexOptions]"Multiline,IgnoreCase,Compiled")
 }
 process {
    if($PSCmdlet.ParameterSetName -eq "File") {
       $AutomaticRecords = $true
       $InputString += $(Get-Content $FilePath -Delimiter ([char]0)) + "`n`n"
    } else {
       $InputString += "$InputObject`n"
    }
 }
 end {
    if(!"$ReRecordSeparator"){
       Write-Verbose "Setting up the record regex in the PROCESS block: '$RecordSeparator'"
       [Regex]$ReRecordSeparator   = New-Object Regex ([System.String]"\s*$RecordSeparator\s*"),   ([System.Text.RegularExpressions.RegexOptions]"Multiline,IgnoreCase,Compiled")
    }
    if(!"$RePropertySeparator"){
       Write-Verbose "Setting up the property regex in the PROCESS block: '$PropertySeparator'"
       [Regex]$RePropertySeparator = New-Object Regex ([System.String]"\s*$PropertySeparator\s*"), ([System.Text.RegularExpressions.RegexOptions]"Multiline,IgnoreCase,Compiled")
    }
    if(!"$ReValueSeparator") {  
       Write-Verbose "Setting up the value regex in the PROCESS block: '$ValueSeparator'"
       [Regex]$ReValueSeparator    = New-Object Regex ([System.String]"\s*$ValueSeparator\s*"),    ([System.Text.RegularExpressions.RegexOptions]"Multiline,IgnoreCase,Compiled")
    }
    Write-Verbose "ParameterSet: $($PSCmdlet.ParameterSetName)"
    Write-Verbose "ValueSeparator: $($ReValueSeparator)"
    $InputData = @{}
    if($PSCmdlet.ParameterSetName -eq "File") {
       $AutomaticRecords = $true
       $InputString = Get-Content $FilePath -Delimiter ([char]0)
    }
    
    if($PsBoundParameters.ContainsKey('RecordSeparator') -or $AutomaticRecords ) {
       if(!@($Records)[0]) {
          $Records = $Records[1..$($Records.Count-1)]
       }
       Write-Verbose "There are $($ReRecordSeparator.GetGroupNumbers().Count) groups and $(@($Records).Count) records!"
       if($ReRecordSeparator.GetGroupNumbers().Count -gt 1 -and @($Records).Count -gt 1) {
          if($HasHeader) {
             $Records = $Records[2..$($Records.Count -1)]
          }
          if($HasFooter) {
             $Records = $Records[0..$($Records.Count -3)]
          }
          while($Records) {
             $Key,$Value,$Records = @($Records)
             Write-Verbose "RecordSeparator with grouping: $Key = $Value"
             $InputData.$Key += @($Value)
          }
       } elseif(@($Records).Count -gt 1) {
          $InputData."" = @($Records)
          $InputString = $Records
       } else {
          $InputString = $Records
       }
    }
       
    if($PsBoundParameters.ContainsKey('CountOfPropertiesPerRecord')) {   
       $Properties = $RePropertySeparator.Split($InputString)
       Write-Verbose "Separating Records by Property count = $CountOfPropertiesPerRecord of $($Properties.Count)"
       for($Index = 0; $Index -lt $Properties.Count; $Index += $CountOfPropertiesPerRecord) {
          $InputData."" += @($Properties[($Index..($Index+$CountOfPropertiesPerRecord-1))] -Join ([char]0))
          Write-Verbose "Record ($Index..) $($Index/$CountOfPropertiesPerRecord) = $(@($Properties[($Index..($Index+$CountOfPropertiesPerRecord-1))] -Join ([char]0)))"
       }
       $SetPropertySeparator = $RePropertySeparator
       [Regex]$RePropertySeparator = New-Object Regex ([System.String][char]0), ([System.Text.RegularExpressions.RegexOptions]"Multiline,IgnoreCase,Compiled")
    } 
    if($InputData.Keys.Count -eq 0){
       Write-Verbose "Keyless entry enabled!"
       $InputData."" = @($InputString)
    }
    
    Write-Verbose "InputData: $($InputData.GetEnumerator() | ft -auto -wrap| out-string)"
 
    foreach($key in $InputData.Keys) { foreach($record in $InputData.$Key) {
       Write-Verbose "Record($Key): $record"
       
       $output = new-output -SimpleOutput:$SimpleOutput -Key:$Key -FilePath:$FilePath
       
       foreach($Property in $RePropertySeparator.Split("$record") | ? {$_}) {
          if($ReValueSeparator.IsMatch($Property)) {
             [string[]]$data = $ReValueSeparator.split($Property,2) | foreach { $_.Trim() } | where { $_ }
             Write-Verbose "Property: $Property --> $($data -join ': ')"
             if($AutomaticRecords -and $Output.ContainsKey($Data[0])) {
                out-output $output
                $output = new-output -SimpleOutput:$SimpleOutput -Key:$Key -FilePath:$FilePath
             }
             switch($data.Count) {
                1 { $output.($Data[0]) += @($null)    }
                2 { $output.($Data[0]) += @($Data[1]) }
             }
          } elseif($AllowEmptyValue) {
             $output.$Property = $true
          }
       }
       out-output $output
 
       
    }  }
    $RePropertySeparator = $SetPropertySeparator
 }
 #}
`

