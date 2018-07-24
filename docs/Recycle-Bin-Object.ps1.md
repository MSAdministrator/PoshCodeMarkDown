---
Author: robespierre
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5267
Published Date: 2017-06-28t11
Archived Date: 2017-05-25t20
---

# recycle bin object - 

## Description

correctly works with reparse points

## Comments



## Usage



## TODO



## function

`calculate-items`

## Code

`#
 #
 $method = @{}
 $method['clear'] = {
 	$this.Items() | foreach -Process {
 		ls $_.Path -Recurse -Force -Directory | foreach {
 			$d_item = gi -LiteralPath $_.FullName
 		}
 	}
 $method['count'] = {
 	Set-Variable -Name links,files,folders,size -Value 0 -Option AllScope
 	function Calculate-Items ($ItemsList)
 	{
 		function count ($folder, $list)
 		{
 			if (-not $list) { $list = Get-Item "$($folder)\*" }
 			$list | foreach {
 				if ($_.Attributes -match 'ReparsePoint') { ++$links }
 				elseif ($_.Attributes -notmatch 'Directory') { ++$files;  $size += $_.Length }
 			}
 		}
 	$upper_items_list = @()
 	$this.Items() | foreach -Process { $upper_items_list += gi -LiteralPath $_.Path }
 	if ($upper_items_list) { Calculate-Items -ItemsList $upper_items_list }
 	return New-Object -TypeName psobject -Property @{
 			Links = $links; Files = $files; Folders = $folders; Length = $size
 	}
 $bin = (New-Object -ComObject 'Shell.Application').Namespace(0xA)
 $bin | Add-Member -MemberType 'ScriptMethod' -Name 'Clear' -Value $method.clear
 $bin | Add-Member -MemberType 'ScriptMethod' -Name 'Count' -Value $method.count
 New-Variable -Name RecycleBin -Value $Bin -Option Constant
 Remove-Variable -Name method, bin
`

