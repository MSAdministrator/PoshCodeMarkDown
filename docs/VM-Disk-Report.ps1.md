---
Author: jgrote
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5148
Published Date: 2014-05-06t04
Archived Date: 2014-05-15t05
---

# vm disk report - 

## Description

gets all virtual machines, and exports a csv that shows their virtual disk capacities and type. used in this case for sizing a vcb temp disk.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 $VMs = get-vm
 $Results = @()
 foreach ($VM in $VMs) {
     $Result = new-object PSObject
     $Result | add-member -membertype NoteProperty -name "Name" -value $VM.Name
     $Result | add-member -membertype NoteProperty -name "Description" -value $VM.Notes
     $VMDiskCount = 1
     get-harddisk $VM | foreach {
         $disk = $_
         $Result | add-member -name "Disk($VMDiskCount)SizeGB" -value ([math]::Round($disk.CapacityKB / 1MB)) -membertype NoteProperty
         $Result | add-member -name "Disk($VMDiskCount)Type" -value $disk.DiskType -membertype NoteProperty
         $VMDiskCount++
     }
     $Results += $Result
 }
 $Results | select-object * | export-csv -notypeinformation E:\VCBDiskReport.csv
`

