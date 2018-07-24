---
Author: paul wiegmans
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 3922
Published Date: 2014-01-30t11
Archived Date: 2014-05-23t00
---

# query magisterwebservice - 

## Description

en

## Comments



## Usage



## TODO



## script

`write-log`

## Code

`#
 #
 <#
 	.TITLE
 		Haal-WisMasLeerling.ps1
 	.AUTHOR
 		Paul Wiegmans  (p [dot] wiegmans [at] bonhoeffer [dot] nl)
 	.DATE
 		2013-01-30  
 	.DESCRIPTION
 		Send a webquery to Schoolmaster Magister web service and request data 
 		from the list WISMASLEERLING. 
 		Doe een webquery naar Magister webservice en haal de lijst WISMASLEERLING op.
 	.NOTES
 	.LINK
 		http://ict.bonhoeffer.nl
 #>
 
 Import-Module ActiveDirectory
 Set-StrictMode -Version 2
 
 $mijnpad = Split-Path -parent $MyInvocation.MyCommand.Definition
 $mijnbasename = $([System.IO.Path]::GetFileNameWithoutExtension($MyInvocation.MyCommand.Definition))
 
 
 $aantal = 0
 
 $SOAPLayout = "WISMASLEERLINGEN"
 $SOAPusername = "username"
 $SOAPpasswd = "password"
 
 Function Write-Log ($tekst) {
 	$logtekst = ("{0:0###}: {1}" -f ($aantal, $tekst))
 	if ($outputting) {	
 		Write-Host $logtekst
 	}
 	if ($Logging) {
 		#$logtekst | Out-File "$mijnpad\$mijnbasename.log" -Append 
 	}
 }
 
 function Doe-Webquery ([string]$Url, [System.Text.Encoding]$encoding = [System.Text.Encoding]::UTF7) {
 	
 	$request = [System.Net.WebRequest]::Create($Url)
 	try {
 		$response = $request.GetResponse()
 	}
 	catch [Net.WebException] {
     	Write-Host $_.Exception.ToString()
 		exit
 	}
 
 	$requestStream = $response.GetResponseStream()
 	$readStream = new-object System.IO.StreamReader( $requestStream, $Encoding )
 
 
 	$readStream.Close()
 	$response.Close()
 	return $result
 }
 
 Function Ophalen-Magister () {
 	$SOAPUrl = "https://bonhoeffer.swp.nl:8800/doc?Function=GetData"`
 		+"&Library=Data&SessionToken=$SOAPusername%3B$SOAPpasswd&Layout=$SOAPlayout"`
 		+"&Parameters=leerjaar%3D1213&Type=CSV"
 	$iso88591 = [System.Text.Encoding]::GetEncoding("ISO-8859-1")
 	$UTF7 = [System.Text.Encoding]::UTF7
 	$data = Doe-Webquery $SOAPUrl	$UTF7
     
     if ($data -is [string]) {
         if (($data.contains("Fout")) -or ($data.contains('ResultMessage'))) {
 			$data = Doe-Webquery $SOAPUrl $iso88591 
 			Write-Log $SOAPUrl
             Throw "Webquery retourneert fout (1): $data"
         }
     }                 
         
 	if ($data[0] -is [string]) {
         if ($data[0].contains("Fout_nummer") ) {
             Write-Log $SOAPUrl
     		Throw "Webquery retourneert fout (2): $data"
         }            
 	}
 	return  ($data | Out-String) -replace "`r`r","`r"
 }
 
 function VerwijderVreemdeTekens($n) {
 	$VervangTabel = @(`
 	"ñn","êe","ëe","èe","ée","áa","äa","öo","óo","ïi","úu","ûu","çc","�S","�s","�s","ýy","-","'")
 	foreach ($letter in $VervangTabel) {
 		$n = $n -replace $letter[0], $letter[1]
 	}
 	return $n
 }
 
 Function VertaalTekens ($n) {
 		voor speciaal geval �romofský . Dit doet een eerst conversie van vreemde tekens 
 		uit de data in ANSI-codering naar herkenbare vreemde tekens, zodat deze later 
 		vervangen, door romeinse tekens. Leer de code door uitvoer "onvertaald.txt" 
 		te zien in hexeditor XVI32.
 	#>
 	return $n -replace [char]138, "�" -replace [char]0x9a, "�"
 }
 
 
 	
 Write-Host " "
 Write-Log "Start --- "
 
 
 $onvertaald = Ophalen-magister
 $onvertaald | Out-File "$mijnpad\onvertaald.txt"
 
 $csv | Export-CSV -Path "$mijnpad\wismasleerling.csv" -Encoding UTF8 -NoTypeInformation
 Write-Log ("Gegevens in webquery: {0}" -f $csv.count)
 if (($csv.Count -lt 1490) -or ($csv.Count -ge 1570)) {
 	throw "Webquery geeft onwaarschijnlijk aantal resultaten. Uitvoering stopt."
 }
 $csv | Out-GridView; exit
`

