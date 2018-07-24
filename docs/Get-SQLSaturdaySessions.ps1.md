---
Author: eli w-h tarwn
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4034
Published Date: 2013-03-21t10
Archived Date: 2013-03-27t04
---

# get-sqlsaturdaysessions - 

## Description

given a sql saturday session number, will retrieve and parse the sessions from the corresponding sql saturday event page. if you provide a value for the $getunscheduled attribute, it retrieves the unscheduled sessions, otherwise the default is to retrieve the scheduled ones.

## Comments



## Usage



## TODO



## function

`get-sqlsaturdaysessionlist`

## Code

`#
 #
 function Get-SQLSaturdaySessionList { 
         param(
         [string] $Number =  "111",
         $GetUnscheduled
         )
         $baseUrl = "http://sqlsaturday.com/" + $Number
         $url = $baseUrl + "/schedule.aspx"
         $page = $null  
         try { 
             Write-Verbose "Initiating WebClient" 
             $webclient = New-Object system.net.webclient 
              
             Write-Verbose "Attempting to download from $url" 
             $page = $webclient.DownloadString($url) 
         } 
         catch [Exception] { 
             Write-Error ("Could not download {0}.  The following error was logged:`r`n{1}" -f $url, $error[0]) 
         }  
          
         if ($page) { 
                 Write-Verbose "Page downloaded" 
         
                 if(!$GetUnscheduled){
                         Write-Verbose "Getting Scheduled" 
                         #<td><a href="/viewsession.aspx?sat=94&amp;sessionid=5471">Randy Knight<br><br>But it worked great in Dev!  Performance for Devs<br><br>Level: Beginner</a></td>
                         #<td><a href="/viewsession.aspx?sat=89&sessionid=4942">Adam Jorgensen<br /><br />Zero to Cube<br/ ><br />Level: Intermediate</a></td>
                         $regex = [regex]'href="(?<url>/viewsession.aspx\?sat=\d+\&(?:amp;)?sessionid=\d+)">\s*(?<speaker>[^<]+)<br\s*/?\s*><br\s*/?\s*>(?<title>[^<]+)<br\s*/?\s*><br\s*/?\s*>Level:\s*(?<level>[^<]+)</a>' 
                 }
                 else{
                         Write-Verbose "Getting Unscheduled" 
                         $regex = [regex]'href="(?<url>/viewsession.aspx\?sat=\d+\&(?:amp;)?sessionid=\d+)">\s*(?<title>[^<]+)</a></td><td>(?<speaker>[^<]+)</td><td>(?<level>[^<]+)</td>' 
                 }
                 
                 $matches = $regex.Matches($page)
 
                 Write-Verbose "Match Count: $($matches.count)"
 
 
                 $table = New-Object system.Data.DataTable "MyTable"
                 $col = New-Object system.Data.DataColumn Title,([string])
                 $table.columns.add($col)
                 $col = New-Object system.Data.DataColumn Speaker,([string])
                 $table.columns.add($col)
                 $col = New-Object system.Data.DataColumn Level,([string])
                 $table.columns.add($col)
                 $col = New-Object system.Data.DataColumn URL,([string])
                 $table.columns.add($col)
 
                 $matches | % {$row = $table.NewRow(); $row.URL = $baseUrl + $_.Groups['url']; $row.Title = $_.Groups['title']; $row.Speaker = $_.Groups['speaker'] -replace '[\s\.,]+',' '; $row.Level = $_.Groups['level']; $table.Rows.Add($row) }
 
                 $table
         } 
 }
 
`

