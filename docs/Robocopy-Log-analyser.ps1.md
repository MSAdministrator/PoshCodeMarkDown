---
Author: bart lievers
Publisher: 
Copyright: 
Email: 
Version: 165.24
Encoding: ascii
License: cc0
PoshCode ID: 5777
Published Date: 2017-03-08t20
Archived Date: 2017-05-22t01
---

# robocopy log analyser - 

## Description

a function to analyse robocopy logs which are placed in a folder.

## Comments



## Usage



## TODO



## function

`analyse-rc_log`

## Code

`#
 #
 function Analyse-RC_Log {
     <#
     .SYNOPSIS
         Robocopy log analyser
     .DESCRIPTION
         analysing the robocopy logs that are generated with the /LOG option.
         It has two modes, anaylsing the summary of log files and analyse the full log.
         The report is saved as a CSV file.
         Returns an custom object containing a RC summary like property, a CSV property.
         During script process the Culture of the script is changed to en-US for date and number interpretation / calculations
 
         Script is based on Robocopy log analyser from http://www.chapmancentral.co.uk/cloudy/2013/02/23/parsing-robocopy-logs-in-powershell/
     .EXAMPLE
         >Analyse-RC_Log -SourcePath d:\RC_logdir -ExcelCSV -fp -unitsize GB
         Analyse log files in d:\RC_logdir, use a semicolon in the CSV file (conform MS Excel). Parse the complete log files and report al Bytes sizes as GB
     .EXAMPLE
         >$Result=Analyse-RC_Log -SourcePath d:\RC_logdir -ExcelCSV -fp -unitsize GB
 	>& $Result.ReportFileName
 	>$Result.Summary | out-gridview
 	
 	opens the CSV file and show the summary in a powershell gridview
     .PARAMETER fp
         File Parsing. Parse the complete file instead of the heather and footer
     .PARAMETER SourcePath
         Path where the Robocopy logs are saved.
     .PARAMETER ExcelCSV
         Use a semicolon as seperator
     .Parameter UnitSize
         Report al Byte sizes  in given unit size.
     .Link
         http://www.chapmancentral.co.uk/cloudy/2013/02/23/parsing-robocopy-logs-in-powershell/
     .NOTES
         File Name      : Analyse-RC_Log.ps1
         Author         : B. Lievers
         Prerequisite   : PowerShell V2 over Vista and upper.
         Copyright 2015 - Bart Lievers
     #>
     [CmdletBinding()]
 
     param(
 	    [parameter(Position=0,Mandatory=$true,ValueFromPipeline=$false,HelpMessage='Source Path with no trailing slash')][string]$SourcePath,
 	    [switch]$fp,
         [Switch]$ExcelCSV,
         [Parameter(HelpMessage='Unit size, default is Bytes when parameter is not present')][ValidateSet("B","GB","KB","MB","TB")][string]$UnitSize
 	    )
  
     begin {
         [System.Globalization.CultureInfo]$culture=[System.Globalization.CultureInfo]("en-US")
         $OldCulture = [System.Threading.Thread]::CurrentThread.CurrentCulture
         trap 
         {
             [System.Threading.Thread]::CurrentThread.CurrentCulture = $OldCulture
         }
         [System.Threading.Thread]::CurrentThread.CurrentCulture = $culture
         Write-Verbose ("Changing Locale from "+$oldCulture.Name+" to "+$culture.Name)
 
         write-host "Robocopy log parser. $(if($fp){"Parsing file entries"} else {"Parsing summaries only, use -fp to parse file entries"})"
  
         $ElapsedTime = [System.Diagnostics.Stopwatch]::StartNew()
 
         $HeaderParams = @{
 	        "04|Started" = "date";	
 	        "01|Source" = "string";
 	        "02|Dest" = "string";
 	        "03|Options" = "string";
 	        "07|Dirs" = "counts";
 	        "08|Files" = "counts";
 	        "09|Bytes" = "counts";
 	        "10|Times" = "counts";
 	        "05|Ended" = "date"
 	        #"06|Duration" = "string"
         }
         #-- summary fields for file tag statistics during file parsing, swich -fp
         $fileTags=@{ 
             "01|MISMATCH" = ""
             "02|EXTRA file" = ""
             "03|New File" = ""
             "04|lonely" = ""
             "05|Newer" = ""
             "06|Newer XN" = ""
             "07|Older" = ""
             "08|Older XO" = ""
             "09|Changed" = ""
             "10|Changed XC" = ""
             "11|Tweaked" = ""
             "12|Same IS" = ""
             "13|Same" = ""
             "14|attrib" = ""
             "15|named" = ""
             "16|large" = ""
             "17|small" = ""
             "18|too old" = ""
             "19|too new" = ""
             "20|New Dir"= ""
         }
         #-- summary fields for directory tag statistics during file parsing, swich -fp
         $DirTags=@{ 
             "01|MISMATCH" = ""
             "02|Extra Dir" = ""
             "03|New Dir" = ""
             "04|lonely" = ""
             "05|named" = ""
             "06|junction" = ""
             "07|exist" = ""
         } 
 
         $ProcessCounts = @{
 	        "Processed" = 0;
 	        "Error" = 0;
 	        "Incomplete" = 0
         }
 
         #-- Default the CSV delim is a comma, when using CSV for Excel a semicolon is needed as delimmiter
         if ($ExcelCSV) { $Delim=";"} else {$Delim=","}
          #-- ASCII tab character
         $tab=[char]9
 
          #-- Get list of files to analyse
         $files=get-childitem $SourcePath
         #-- let's start writing, shall we ? 
         $writer=new-object System.IO.StreamWriter("$(get-location)\robocopy-$(get-date -format "dd-MM-yyyy_HH-mm-ss").csv")
 
 
                                                                                                                                     
         function Get-Tail{
             <#
             .SYNOPSIS
                 Get tail of file
             .EXAMPLE
                 >Get-Tail $reader 20
             .PARAMETER reader
                 an IO stream file object
             .PARAMETER count
                 Number of rows to collect
             #>
             Param (
                 [object]$reader, 
                 [int]$count = 10
                 )
 
 	        $lineCount = 0
 	        [long]$pos = $reader.BaseStream.Length - 1
  
 	        while($pos -gt 0) {
 		        $reader.BaseStream.position=$pos
  
 		        if ($reader.BaseStream.ReadByte() -eq 10) {
 			        $lineCount++
 			        if ($lineCount -ge $count) { break }
 		            }
 		        $pos--
 	            } 
  
 	        if ($lineCount -lt $count -or $pos -ge $reader.BaseStream.Length - 1) {
 		        $reader.BaseStream.Position=0
 	        } else {
 	        }
  
 	        $lines=@()
 	        while(!$reader.EndOfStream) {
 		        $lines += $reader.ReadLine()
 	        }
 	        return $lines
         }
  
         function Get-Top {
             <#
             .SYNOPSIS
                 Get top of file
             .EXAMPLE
                 >Get-Top $reader 20
             .PARAMETER reader
                 an IO stream file object
             .PARAMETER count
                 Number of rows to collect
             #>
             Param(
                 [object]$reader,
                 [int]$count = 10
             )
 
 	        $lines=@()
 	        $lineCount = 0
 	        $reader.BaseStream.Position=0
 	        while(($linecount -lt $count) -and !$reader.EndOfStream) {
 		        $lineCount++
 		        $lines += $reader.ReadLine()		
 	        }
 	        return $lines
         }
  
         function Remove-Key{
             <#
             .SYNOPSIS
                 Return the name without the ID
             .EXAMPLE
                 >Remove-Key -name "01|example"
             .PARAMETER name
                 a string where the ID needs to be stripped
             #>
             Param ( $name 
             )
 	        if ( $name -match "|") {
 		        return $name.split("|")[1]
 	        } else {
 		        return ( $name )
 	        }
         }
  
         function Get-Value{
             <#
             .SYNOPSIS
                 Get the value of a RC line
             .EXAMPLE
                 >Get-Value -line " filecount : 35555" -variable "Filecount"
                 Returns 35555
             .PARAMETER line
                 A RC log string
             .PARAMETER variable
                 The variable that needs to be extracted
             #>
             Param(
                 $line,
                 $variable
             )
 	        if ($line -like "*$variable*" -and $line -like "* : *" ) {
 		        $result = $line.substring( $line.IndexOf(":")+1 )
 		        return $result 
 	        } else {
 		        return $null
 	        }
         }
  
         function UnBodge-Date{
             <#
             .SYNOPSIS
                 Convert Robocopy date to a usable format
             .EXAMPLE
                 >UnBodge-Date -dt "Sat Feb 16 00:16:49 2013"
                 Returns 16-02-2013 00:16:49 in Locale format
             .PARAMETER dt
             #>
             Param(
                 $dt
             )
 	        if ( $dt -match ".{3} .{3} \d{2} \d{2}:\d{2}:\d{2} \d{4}" ) {
 		        $dt=$dt.split(" ")
                 $dt=$dt[2]+"/"+$dt[1]+"/"+$dt[4],$dt[3]
 		        $dt -join " " | Out-Null
 	        }
 	        if ( $dt -as [DateTime] ) {
                 return(get-date $dt -format "dd/MM/yyy HH:mm:ss")
 	        } else {
 		        return $null
 	        }
         }
  
         function Unpack-Params{
             <#
             .SYNOPSIS
 	            Unpacks file count bloc in the format
 	             Dirs :      1827         0      1827         0         0         0
 	            Files :      9791         0      9791         0         0         0
 	            Bytes :  165.24 m         0  165.24 m         0         0         0
 	            Times :   1:11:23   0:00:00                       0:00:00   1:11:23
 	            Parameter name already removed
             .EXAMPLE
                 >UnBodge-Date -dt "Sat Feb 16 00:16:49 2013"
                 Returns 16-02-2013 00:16:49 in Locale format
             .PARAMETER params
                 
             #>
             Param(
                 $params
             )
  
 	        if ( $params.length -ge 58 ) {
 		        $params = $params.ToCharArray()
 		        $result=(0..5)
 		        for ( $i = 0; $i -le 5; $i++ ) {
 			        $result[$i]=$($params[$($i*10 + 1) .. $($i*10 + 9)] -join "").trim()
 		        }
 		        #$result=$result -join ","
                 $result=$result -join $Delim
 	        } else {
 		        #$result = ",,,,,"
                 $result=$Delim+$Delim+$Delim+$Delim+$Delim
 	        }
 	        return $result
         }
     } #-- end of Begin
     
     Process{
     $sourcecount = 0
         $targetcount = 1
 
     $writer.Write("File")
     $fields="File"
     foreach ( $HeaderParam in $HeaderParams.GetEnumerator() | Sort-Object Name ) {
 	    if ( $HeaderParam.value -eq "counts" ) {
             $tmp="~ Total"+$Delim+"~ Copied"+$Delim+"~ Skipped"+$Delim+"~ Mismatch"+$Delim+"~ Failed"+$Delim+"~ Extras"
             if ((Remove-Key $headerparam.name) -match "Bytes") {
                 #-- if column header is a Bytes Column then match it to the unitsize
                 if ($UnitSize -and $UnitSize -ne "B") {
                     #-- change the Bytes header according to the unitsize, Unitsize is GB ==> header is GBytes
 		            $tmp=$tmp.replace("~",$UnitSize.Substring(0,1)+ "$(Remove-Key $headerparam.name)")
                 } else {
                     $tmp=$tmp.replace("~","$(Remove-Key $headerparam.name)")
                 }
             } else {
 		        $tmp=$tmp.replace("~","$(Remove-Key $headerparam.name)")
             }
             $fields=$fields+$Delim+"$($tmp)"
 		    $writer.write($Delim+"$($tmp)")
 	    } else {
             $writer.write($Delim+"$(Remove-Key $HeaderParam.name)")
             $fields=$fields+$Delim+"$(Remove-Key $HeaderParam.name)"
 	    }
     }
  
     if($fp){
         $writer.write($Delim+"Scanned"+$Delim+"Newest"+$Delim+"Oldest")
         $fields=$fields+$Delim+"Scanned"+$Delim+"Newest"+$Delim+"Oldest"
         foreach ($fileTag in $filetags.GetEnumerator() | Sort-Object Name ) {
             $writer.write($Delim+$filetag.name.split("|")[1])
              $fields=$fields+$delim+$filetag.name.split("|")[1]
                 }
         foreach ($DirTag in $Dirtags.GetEnumerator() | Sort-Object Name ) {
             $writer.write($Delim+"DIR "+$DirTag.name.split("|")[1])
              $fields=$fields+$delim+"DIR "+$DirTag.name.split("|")[1]
         }
     }
     $writer.WriteLine()
 
         $table=$fields
         $filecount=0
  
         foreach ($file in $files) {  
 	        $filecount++
             write-host "$filecount/$($files.count) $($file.name) ($($file.length) bytes)"
 	        $results=@{}
             #-- open the file
 	        $Stream = $file.Open([System.IO.FileMode]::Open, 
                            [System.IO.FileAccess]::Read, 
                             [System.IO.FileShare]::ReadWrite) 
 	        $reader = New-Object System.IO.StreamReader($Stream) 
  
 	        $HeaderFooter = Get-Top $reader 16
  
 	        if ( $HeaderFooter -match "ROBOCOPY     ::     Robust File Copy for Windows" ) {
                 #-- file has valid header
 		        if ( $HeaderFooter -match "Files : " ) {
 			        $HeaderFooter = $HeaderFooter -notmatch "Files : "
 		        }
  
 		        [long]$ReaderEndHeader=$reader.BaseStream.position
             
 		        $Footer = Get-Tail $reader 16
 		        $ErrorFooter = $Footer -match "ERROR \d \(0x000000\d\d\) Accessing Source Directory"
 		        if ($ErrorFooter) {
 			        $ProcessCounts["Error"]++
 			        write-host -foregroundcolor red "`t $ErrorFooter"
 		        } elseif ( $footer -match "---------------" ) {
 			        $ProcessCounts["Processed"]++
 			        $i=$Footer.count
 			        while ( !($Footer[$i] -like "*----------------------*") -or $i -lt 1 ) { $i-- }
 			        $Footer=$Footer[$i..$Footer.Count]
 			        $HeaderFooter+=$Footer
 		        } else {
 			        $ProcessCounts["Incomplete"]++
                     write-warning "`t Log file $file is missing the footer and may be incomplete"
 		        }
  
 		        foreach ( $HeaderParam in $headerparams.GetEnumerator() | Sort-Object Name ) {
 			        $name = "$(Remove-Key $HeaderParam.Name)"
 			        $tmp = Get-Value $($HeaderFooter -match "$name : ") $name
 			        if ( $tmp -ne "" -and $tmp -ne $null ) {
 				        switch ( $HeaderParam.value ) {
 					        "date" { $results[$name]=UnBodge-Date $tmp.trim() }
 					        "counts" { $results[$name]=Unpack-Params $tmp }
 					        "string" { $results[$name] = """$($tmp.trim())""" }		
 					        default { $results[$name] = $tmp.trim() }		
 				        }
 			        }
                     #-- convert bytes statistics to numbers
                     if ($name -eq "Bytes" -and ($results[$name] -ne $null)) {
                         $tmp=@()     
                         $results[$name].split($Delim) | % {
                             #-- convert value to MBytes 
                             $Bytes=$_
                             if ($Bytes -match "m|g|t|k") {
                                 switch ($Bytes.split(" ")[1]) {                
                                     "m" {$conv=1MB*$Bytes.replace(" m","MB")/1MB}
                                     "k" {$conv=1KB*$Bytes.replace(" k","KB")/1KB}
                                     "g" {$conv=1GB*$Bytes.replace(" g","GB")/1GB}
                                     "t" {$conv=1TB*$Bytes.replace(" t","TB")/1TB}
                                 }
                             } else { 
                                 #-- copy original value, no unit sign detected
                                 $conv = $Bytes
                             }
                             #-- convert size 
                             switch ($UnitSize) {
                                 "MB" {$tmp+=($conv/1MB)}
                                 "KB" {$tmp+=($conv/1KB)}
                                 "GB" {$tmp+=($conv/1GB)}
                                 "TB" {$tmp+=($conv/1TB)}
                                 default {$tmp+=($conv)} #-- no conversion needed. Size is in Bytes
                             }              
                         }
                         #-- rebuild string
                         $results[$name]=$tmp -join $delim                    
                     } #-- end of converting bytes statistics          
 		        } #-- end of parsing headerparam fields
  
 
 		        if ( $fp ) {
                     #-- parse the complete file
 			        write-host "Parsing $($reader.BaseStream.Length) bytes" -NoNewLine
  
                     $FT_counters=@{}
                     $DT_counters=@{}
                     $FileTags.GetEnumerator() | select name | %{ $FT_counters.add($_.name.split("|")[1],0)}
                     $DirTags.GetEnumerator() | select name | %{ $DT_counters.add($_.name.split("|")[1],0)}
 			        $reader.BaseStream.Position=0
 			        $filesdone = $false
 			        $linenumber=0
 			        $FileResults=@{}
 			        $newest=[datetime]"1/1/1900"
                     $oldest=Get-Date
 			        $linecount++
 			        $firsttick=$elapsedtime.elapsed.TotalSeconds
 			        $tick=$firsttick+$refreshrate
 			        $LastLineLength=1
  
 			        try {
 				        do {
 					        $line = $reader.ReadLine()
 					        $linenumber++
 					        if (($line -eq "-------------------------------------------------------------------------------" -and $linenumber -gt 16)  ) { 
 						        $filesdone=$true
 					        } elseif ($linenumber -gt 16 -and $line -gt "" ) {
                                 #-- split line according to TAB
 						        $buckets=$line.split($tab)
 
 						        if ( $buckets.count -gt 3 ) {
                                     #-- line contains file information
 							        $status=$buckets[1].trim()
 							        $FileResults["$status"]++
                                     #-- File tag counters
                                     if ($status -ceq "Newer") { $FT_counters["$status" +" XN"]++ }
                                     elseif ($status -ceq "Older") {  $FT_counters["$status" +" XO"]++ } 
                                     elseif ($status -ceq "Changed") {  $FT_counters["$status" +" XC"]++ }
                                     elseif ($status -ceq "same") {  $FT_counters["$status" +" IS"]++ }
                                     else {$FT_counters["$status"]++}
 
                                     #-- Get Timestamp from file
 							        $SizeDateTime=$buckets[3].trim()
 							        if ($sizedatetime.length -gt 19 ) {
 								        $DateTime = $sizedatetime.substring($sizedatetime.length -19)
 								        if ( $DateTime -as [DateTime] ){
 									        $DateTimeValue=[datetime]$DateTime
 									        if ( $DateTimeValue -gt $newest ) { $newest = $DateTimeValue }
 									        if ( $DateTimeValue -lt $oldest ) { $oldest = $DateTimeValue }
 								        }
 							        }
 						        } elseif ($buckets.count -eq 3) {
                                     #-- line contains directory information
                                     #-- Directory tag counters
                                     $status=$buckets[1].Substring(0,10).trim()
                                     if ($status.length -gt 0){
                                         $DT_counters["$status"]++
                                     } else {
                                         $DT_counters["Exist"]++
                                     }
                                 }
 					        }
 
                             #-- progress indicator 
 					        if ( $elapsedtime.elapsed.TotalSeconds -gt $tick ) {
 						        $line=$line.Trim()
 						        if ( $line.Length -gt 48 ) {
 							        $line="[...]"+$line.substring($line.Length-48)
 						        }
 						        $line="$([char]13)Parsing > $($linenumber) ($(($reader.BaseStream.Position/$reader.BaseStream.length).tostring("P1"))) - $line"
 						        write-host $line.PadRight($LastLineLength) -NoNewLine
 						        $LastLineLength = $line.length
 						        $tick=$tick+$refreshrate						
 					        }
  
 				        } until ($filesdone -or $reader.endofstream)
 			        }
 			        finally {
 				        $reader.Close()
 			        }
  
 			        $line=$($([string][char]13)).padright($lastlinelength)+$([char]13)
 			        write-host $line -NoNewLine
 		        }
 
                 #-- write results
 		        $writer.Write("`"$file`"")
                 $line="`"$file`""
                 #-- write values
 		        foreach ( $HeaderParam in $HeaderParams.GetEnumerator() | Sort-Object Name ) {
 			        $name = "$(Remove-Key $HeaderParam.Name)"
 			        if ( $results[$name] ) {
                         $writer.Write("$Delim$($results[$name])")
                         $line=$line+"$Delim$($results[$name])"
 			        } else {
 				        if ( $ErrorFooter ) {
 					        #-- placeholder
 				        } elseif ( $HeaderParam.Value -eq "counts" ) {
                             #-- write summary counters
                             $writer.Write($Delim+$Delim+$Delim+$Delim+$Delim+$Delim)
                             $line=$line+"$Delim$($results[$name])"
 				        } else {
 					        $writer.Write($Delim) 
                             $line=$line+"$Delim$($results[$name])"
 				        }
 			        }
 		        }
  
 		        if ( $ErrorFooter ) {
 			        $tmp = $($ErrorFooter -join "").substring(20)
                     $tmp=$tmp.substring(0,$tmp.indexof(")")+1)+$Delim+$tmp
 			        $writer.write($delim+$delim+"$tmp")
                     $line=$line+"$Delim$($results[$name])"
 		        } elseif ( $fp ) {
                     #-- write values from file parsing		
 			        $writer.write($delim+"$LineCount"+$delim+"$($newest.ToString('dd/MM/yyyy hh:mm:ss'))"+$delim+"$($oldest.ToString('dd/MM/yyyy hh:mm:ss'))")		
                     $line=$line+"$Delim$($results[$name])"
                     #-- write File tag counters	
                     foreach ($fileTag in $filetags.GetEnumerator() | Sort-Object Name ) {
                     $writer.write($delim+"$($FT_counters[$Filetag.name.split("|")[1]])")
                     $line=$line+$delim+$($FT_counters[$Filetag.name.split("|")[1]])
                     }
                     #-- write directory tag counters
                     foreach ($DirTag in $DirTags.GetEnumerator() | Sort-Object Name ) {
                     $writer.write($delim+"$($DT_counters[$dirtag.name.split("|")[1]])")
                     $line=$line+$delim+$($dT_counters[$DirTag.name.split("|")[1]])
                     }
 		        }
                 #-- EOL
 		        $writer.WriteLine()
                 $table=$table+"`n"+$line
 	        } else {
                 #-- file is not a RoboCopy log file
                 write-warning "$($file.name) is not recognised as a RoboCopy log file"
 	        }
         }
     } #- -end of Process
 
     End{
         #-- write and output results
         write-host "$filecount files scanned in $($elapsedtime.elapsed.tostring()), $($ProcessCounts["Processed"]) complete, $($ProcessCounts["Error"]) have errors, $($ProcessCounts["Incomplete"]) incomplete"
         write-host  "Results written to $($writer.basestream.name)"
         $CSVFile=$writer.basestream.name
         $writer.close() #-- yes, we are done writing
         #-- create output object, containing name of CSV file and Report object
         $Output=New-Object -TypeName psobject -Property @{ReportFileName=$CSVFile
                                                           Report=ConvertFrom-Csv -Delimiter $Delim -InputObject $table
                                                           Summary=@()}
 
         #-- summize, build a Robocopy like Summary
         $Types=@("Dirs","Files",("Bytes".replace("B",$SizeUnit)),"Times","Speed")
         $types | % {
             $row= "" | select  Type,Total,Copied,Skipped,Mismatch,FAILED,Extras
             $row.type=$_
             if ($row.type -ilike "Times") { 
                 #-- calculate times
                 $Output.Report | %{
                     if ($_.ended.length -gt 0){
                         #-- only use data from complete RC logs
                         $row.total=$row.total+[timespan]$_."Times Total"
                         $row.Failed=$row.Failed+[timespan]$_."Times Failed"
                         $row.Copied=$row.Copied+[timespan]$_."Times Copied"
                         $row.Extras=$row.Extras+[timespan]$_."Times Extras"   
                     } else {
                         Write-Verbose ("RC log "+$_.file+" skipped for summary calculation. Log file is not complete.")
                     }     
                 }
             } elseif ($row.type -ilike "Speed") {
                 #-- Calculate speed
                 $Duration=0
                 $output.Report | %{
                         if ($_.ended.length -gt 0){
                             #-- only use data from complete RC logs
                             $Duration=$Duration+[timespan]$_."Times Copied"
                         }
                     }
                 $row.Copied="{0:N1}" -F ((($output.report | Measure-Object -Property (("Bytes".replace("B",$SizeUnit))+" Copied") -Sum).sum) / $Duration.seconds)
             } else {
                 #-calculate counters
                 $row.Total="{0:N1}" -f ($output.report | Measure-Object -Property ($row.type+" Total") -Sum).sum
                 $row.Copied="{0:N1}" -f($output.report | Measure-Object -Property ($row.type+" Copied") -Sum).sum
                 $row.Skipped="{0:N1}" -f($output.report | Measure-Object -Property ($row.type+" Skipped") -Sum).sum
                 $row.Mismatch="{0:N1}" -f($output.report | Measure-Object -Property ($row.type+" Mismatch") -Sum).sum
                 $row.Failed="{0:N1}" -f($output.report | Measure-Object -Property ($row.type+" Failed") -Sum).sum
                 $row.Extras="{0:N1}" -f($output.report | Measure-Object -Property ($row.type+" Extras") -Sum).sum
             }
             $Output.Summary+=$row
         }
         [System.Threading.Thread]::CurrentThread.CurrentCulture = $OldCulture
         Write-Verbose ("Changed Locale back to "+$oldCulture.Name)
         return($output) #-- done, fini, finaly....
     } #-- end of End
 }
`

