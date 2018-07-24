---
Author: halr9000
Publisher: 
Copyright: 
Email: 
Version: 0.25
Encoding: ascii
License: cc0
PoshCode ID: 3015
Published Date: 2011-10-19t14
Archived Date: 2014-11-07t09
---

# trace-route - 

## Description

posting on behalf of james brundage of http

## Comments



## Usage



## TODO



## function

`trace-route`

## Code

`#
 #
 function Trace-Route
 {
    param(
    [Parameter(Mandatory=$true)]
    [Uri]$Url,
    [Timespan]$Timeout = "0:0:0.25",
    [Int]$MaximumHops = 32
    )
 
    process {
        Invoke-Expression "tracert -w $($timeOut.TotalMilliseconds) -h $MaximumHops $url" |
            Where-Object {
                if ($_ -match "[.+]") {
                    $destination
                    try {
                        $destination = [IpAddress]$_.Split("[]",[StringSplitOptions]"RemoveEmptyEntries")[-1]
                    } catch {
                        return $false
                    }
                }
                $true
            } |
            Where-Object {
                if ($_ -like "*Request timed out.*") {
                    throw "Request timed Out"
                }
                return $true
            } |
            Foreach-Object {
                if ($_ -like "*ms*" ) {
                    $chunks = $_ -split "  " | Where-Object { $_ }
                    $destAndip = $chunks[-1]
                    $dest, $ip = $destAndip.Replace("[", "").Replace("]","") -split " "
 
                    if (-not $ip) {
                        $ip = $dest
                        $dest = ""
                    }
 
                    $ip = @($ip)[0].Trim() -as [IPAddress]
 
 
                    if ($chunks[1] -eq '*' -and $chunks[2] -eq '*' -and $chunks[3] -eq '*') {
                        Write-Error "Request Timed Out"
                        return
                    }
                    $trace = New-Object Object
                    $time1 = try { [Timespan]::FromMilliseconds($chunks[1].Replace("<","").Replace(" ms", ""))} catch {}
                    $time2 = try { [Timespan]::FromMilliseconds($chunks[1].Replace("<","").Replace(" ms", ""))} catch {}
                    $time3 = try { [Timespan]::FromMilliseconds($chunks[1].Replace("<","").Replace(" ms", ""))} catch {}
                    $trace |
                        Add-Member NoteProperty HopNumber ($chunks[0].Trim() -as [uint32]) -PassThru |
                        Add-Member NoteProperty Time1 $time1 -PassThru |
                        Add-Member NoteProperty Time2 $time2 -PassThru |
                        Add-Member NoteProperty Time3 $time3 -PassThru |
                        Add-Member NoteProperty Ip $ip -PassThru |
                        Add-Member NoteProperty Host $dest -PassThru |
                        Add-Member NoteProperty DestinationUrl $url -PassThru |
                        Add-Member NoteProperty DestinationIP $destination -PassThru
 
                }
            }
    }
 }
`

