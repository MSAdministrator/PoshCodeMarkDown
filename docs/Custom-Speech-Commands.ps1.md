---
Author: tylert
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1859
Published Date: 2011-05-17t22
Archived Date: 2011-08-17t08
---

# custom speech commands - 

## Description

pertains to the following

## Comments



## Usage



## TODO



## module

`get-definition`

## Code

`#
 #
 import-module "G:\Documents\Speech Macros\custom.psm1"
 import-module "G:\Documents\Speech Macros\alice.psm1"
 Add-SpeechCommands @{
    "test command" 					= { Say $(Respond "3:2,4:0-2") }
    " * the percentages * " 			= { Say $(Percentages) }
    " * star date  * "		 		= { Say "Current, Star date,$("$((Get-Date).year).$([math]::round($((Get-Date).dayofyear)/365, 2)*100)" -replace "([A-Za-z0-9.]){1}","`$1 " -replace "\.","point")" }
    " * mail * " 					= { Start-Process "https://mail.google.com" }
    "Google * please" 				= { $term = $_ -replace "$Computer(.+?)Google (.+?) please", '$2'; Start-Process "http://www.google.com/search?q=$term" }
    "What * time * " 				= { Say "It is $(Get-Date -f "h:mm tt")" }
    "What is * time  * " 		 	= { Say "It is $(Get-Date -f "h:mm tt")" }
    "What time  * " 					= { Say "It is $(Get-Date -f "h:mm tt")" }
    " * some music * " 				= { Start-Process "grooveshark.exe" }
    "What * date today" 				= { Say $(Get-Date -f "dddd, MMMM dd") }
    "What's running?"  				= {
       $proc = ps | sort ws -desc
       Say $("$($proc.Count) processes, including $($proc[0].name), which is using " +
             "$([int]($proc[0].ws/1mb)) megabytes of memory")
    }
    " * the definition for * please" = {$define = $_ -replace "$Computer(.+?)the definition for (.+?) please", '$2' ;$outtext = Get-Definition("$define");Say "Definitions for $define, , , $outtext"}
 
    } -Computer "alice" -Verbose
 
 
 
 
 function Get-Definition($word) {
 #.Synopsis
 #.Description
 #
 	Say "Sure"
 	if($word -match " "){Say "Fail, one word at a time please"; break}
 	$page = "http://simple.wiktionary.org/wiki/$word"
 	$ie = new-object -com "InternetExplorer.Application"
 	$ie.visible = $false
 	$ie.navigate($page)
     while($ie.busy){Start-Sleep 1}
 	$output = " "
 	$lines = 0
 	
     
        $ol 		= @($ie.Document.getElementsByTagName("ol"))[0]
        $li		= @($ol.getElementsByTagName("li"))[$d]
        $def 	= @($li.innerHTML)
        $output += "$def"
 	   if($d -lt $lines){ $output += ", or, " }
     } 
 	
     $ie.Quit()			
     $closeIE = [System.Runtime.Interopservices.Marshal]::FinalReleaseComObject($ie)	
     Remove-Variable ie  		
 	$Clean-String($output)
 }
 
 function Clean-String([string]$str){
 #.Synopsis
 #.Description
 #
 	$str = $str -replace "\<[^<]*\>", " "
 	$str = $str.replace("&nbsp", "")
 	$str
 }
 
 function Respond($in, [string]$del = ","){
 #.Synopsis
 #.Description
 #
 	switch -wildcard ($del){
         "`,"{
         
                 foreach($coordset in $coordsets){
                     $choices += ,(Respond $coordset "`:")
                 }
             }
                 $choices = ,(Respond $in "`:")       
             } 
             
             
         }
         "`:"{
                 foreach ($coord in $coords){                     
                     }
                     else{
                     }
                 }  
                 
         }
         "`/"{
             foreach($range in $ranges){
                     foreach ($val in (Respond $range "`-")){
                     }
                 }
                     $orVals += ,$range
                 }                  
 			}
             
 		}
         "`-"{
             
 		}
     }
 }
 
 		"Right",
 		"Correct"
 	),
 		"Wrong",
 		"Incorrect"
 	
 	),
 		"Yes",
 		"Aye",
 		"I'll give it a shot"
 	
 	),
 		"No",
 		"Negative",
 		"Not going to happen"
 	
 	),
 		"Sure",
 		"OK",
 		"Lets See"
 		
 	),
 		"your mom",
 		"your face"
 	
 	),
 		"You\'re dumb",
 		"you fail"
 	
 	),
 		"I'm ready",
 		"Lets check it out"
 	
 	),
 		"I don't know",
 		"Just a seck"
 	
 	),
 		"oh",
 		"Hum"
 	
 	)
 )
  
 function Percentages {
 #.Synopsis
 #.Description
 
     
     
         Sunday    { [int]$dayOfWeek="1"; break }
         Monday    { [int]$dayOfWeek="2"; break }
         Tuesday   { [int]$dayOfWeek="3"; break }
         Wednesday { [int]$dayOfWeek="4"; break }
         Thursday  { [int]$dayOfWeek="5"; break }
         Friday    { [int]$dayOfWeek="6"; break }
         Saturday  { [int]$dayOfWeek="7"; break }
     }
     
     
     $percentMonthLeft     = [math]::round((1-$percentMonthGone)*100,1)          #^
     $percentWeekLeft      = [math]::round((1-$percentWeekGone)*100,1)           #^
     $percentDayLeft       = [math]::round((1-$percentDayGone)*100,1)            #^
     
     $str = ""
     $str += "There's "
     $str += $percentYearLeft
     $str += "% of this year, "
     $str += $percentMonthLeft
     $str += "% of this month, "
     $str += $percentWeekLeft
     $str += "% of this week, and "
     $str += $percentDayLeft
     $str += "% of today remaining, what will you do with it?"
     $str
 }
`

