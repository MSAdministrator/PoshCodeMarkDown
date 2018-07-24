---
Author: santiago fernandez mu&#241;oz    #
Publisher: 
Copyright: 
Email: 
Version: 2.25
Encoding: utf-8
License: cc0
PoshCode ID: 3301
Published Date: 2012-03-29t11
Archived Date: 2012-04-06t18
---

# listacl.ps1                  # - 

## Description

this script generate a html report show all acls asociated with a folder tree structure starting in root specified by the user

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #######################################
 #######################################
 	
 param ([string] $computer = 'localhost',
 		[string] $path = $(if ($help -eq $false) {Throw "A path is needed."}),
 		[int] $level = 0,
 		[string] $scope = 'administrator', 
 		[switch] $help = $false,
 		[switch] $debug = $false
 	)
 	
 $allowedLevels = 10
 $Stamp = get-date -uformat "%Y%m%d"
 $report = "$PWD\$computer.html"
 $comparison = ""
 $UNCPath = "\\" + $computer + "\" + $path + "\"
 
 if ($level -gt $allowedLevels -or $level -lt 0) {Throw "Level out of range."}
 if ($computer -eq 'localhost' -or $computer -ieq $env:COMPUTERNAME) { $UNCPath = $path }
 switch ($scope) {
 	micro {
 		$comparison = '($acl -notlike "*administrator*" -and $acl -notlike "*BUILTIN*" -and $acl -notlike "*NT AUTHORITY*")'
 	}
 	user {
 		$comparison = '($acl -notlike "*administrator*" -and $acl -notlike "*BUILTIN*" -and $acl -notlike "*IT*" -and $acl -notlike "*NT AUTHORITY*" -and $acl -notlike "*All*")'
 	}
 }
 
 function drawDirectory([ref] $directory) {
 	$dirHTML = '
 	<div class="'
 		if ($directory.value.level -eq 0) { $dirHTML += 'he0_expanded' } else { $dirHTML += 'he' + $directory.value.level } 
 		$dirHTML += '"><span class="sectionTitle" tabindex="0">Folder ' + $directory.value.Folder.FullName + '</span></div>
 		<div class="container"><div class="he' + ($directory.value.level + 1) + '"><span class="sectionTitle" tabindex="0">Access Control List</span></div>
 			<div class="container">
 				<div class="heACL">
 					<table class="info3" cellpadding="0" cellspacing="0">
 						<thead>
 							<th scope="col"><b>Owner</b></th>
 							<th scope="col"><b>Privileges</b></th>
 						</thead>
 						<tbody>'
 		foreach ($itemACL in $directory.value.ACL) {
 			$acls = $null
 			if ($itemACL.AccessToString -ne $null) {
 				$acls = $itemACL.AccessToString.split("`n")
 			}
 			$dirHTML += '<tr><td>' + $itemACL.Owner + '</td>
 			<td>
 			<table>
 				<thead>
 					<th>User</th>
 					<th>Control</th>
 					<th>Privilege</th>
 				</thead>
 				<tbody>'
 			foreach ($acl in $acls) {
 				$temp = [regex]::split($acl, "(?<!(,|NT))\s{1,}")
 				if ($debug) {
 					write-host "ACL(" $temp.gettype().name ")[" $temp.length "]: " $temp
 				}
 				if ($temp.count -eq 1) {
 					continue
 				}
 				if ($scope -ne 'administrator') {
 					if ( Invoke-Expression $comparison ) {
 						$dirHTML += "<tr><td>" + $temp[0] + "</td><td>" + $temp[1] + "</td><td>" + $temp[2] + "</td></tr>"
 					}
 				} else {
 					$dirHTML += "<tr><td>" + $temp[0] + "</td><td>" + $temp[1] + "</td><td>" + $temp[2] + "</td></tr>"
 				}
 			}
 			$dirHTML += '</tbody>
 						</table>
 						</td>
 						</tr>'
 		}
 $dirHTML += '
 						</tbody>
 					</table>
 				</div>
 			</div>
 		</div>'
 	return $dirHTML
 }
 
 
 if ($help) {
 	Write-Host @"
 /··················································\
 · Script gather access control lists per directory ·
 \··················································/
 
 USAGE: ./listACL -computer <machine or IP> 
                  -path <path>
                  -level <0-10>
                  -help:[$false]
 	
 PARAMETERS:
 	computer [OPTIONAL]     - Computer name or IP addres where folder is hosted (Default: localhost)
 	path [REQUIRED]         - Folder's path to query.
 	level [OPTIONAL]        - Level of folders to go down in the query. Allowd values are between 0 and $allowedLevels.
 	                          0 show that there's no limit in the going down (Default: 0)
 	scope [OPTIONAL]        - Sets the amount of information showd in the report. Allowd values are: 
                                   · user, show important information to the user.
                                   · micro, show user scope information plus important information for the IT Department.
                                   · administrator, show all information.
 	help [OPTIONAL]         - This help.
 "@
 	exit 0
 	
 }
 
 if (Test-Path $report)
  {
   Remove-item $report
  }
 
 if ($path.Substring($path.Length - 1,1) -eq "\") { $path = $path.Substring(0,$path.Length - 1) }
 
 @"
 <html dir="ltr" xmlns:v="urn:schemas-microsoft-com:vml" gpmc_reportInitialized="false">
 <head>
 <meta http-equiv="Content-Type" content="text/html; charset=UTF-16" />
 <title>Access Control List for $path in $computer</title>
 <!-- Styles -->
 <style type="text/css">
 
                 table{ font-size:100%; table-layout:fixed; width:100%; }
 
                 td,th{ overflow:visible; text-align:left; vertical-align:top; white-space:normal; }
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 				
 				
 				
 				
 				
 				
 
                 .he0 .expando{ font-size:100%; }
 
                 .info, .info3, .info4, .disalign{ line-height:1.6em; padding:0px 0px 0px 0px; margin:0px 0px 0px 0px; }
 
                 .disalign TD{ padding-bottom:5px; padding-right:10px; }
 
                 .info TD{ padding-right:10px; width:50%; }
 
                 .info3 TD{ padding-right:10px; width:33%; }
 
                 .info4 TD, .info4 TH{ padding-right:10px; width:25%; }
 				
 				.info5 TD, .info5 TH{ padding-right:0px; width:20%; }
 
 
 
                 .subtable TD, .subtable3 TD{ padding-left:10px; padding-right:5px; padding-top:3px; padding-bottom:3px; line-height:1.1em; width:10%; }
 
 
 
 
 
                 .subtable_frame TD{ line-height:1.1em; padding-bottom:3px; padding-left:10px; padding-right:15px; padding-top:3px; }
 
 
 
 
 
 
 
                 .container{ display:block; position:relative; }
 
 
 
 
 
 
 
 
 
 
 </style>
 </head>
 <body>
 <table class="title" cellpadding="0" cellspacing="0">
 <tr><td colspan="2" class="gponame">Access Control List for $path on machine $computer</td></tr>
 <tr>
     <td id="dtstamp">Data obtained on: $(Get-Date)</td>
     <td><div id="objshowhide" tabindex="0"></div></td>
 </tr>
 </table>
 <div class="filler"></div>
 "@ | Set-Content $report
 
 $colFiles = Get-ChildItem -path $UNCPath -Filter *. -Recurse -force -Verbose | Sort-Object FullName
 $colACLs = @()
 foreach($file in $colFiles)
 {
 $matches = (([regex]"\\").matches($file.FullName.substring($path.length, $file.FullName.length - $path.length))).count
 if ($level -ne 0 -and ($matches - 1) -gt $level) {
 	continue
 }
 if ($debug) {
 	Write-Host $file.FullName "->" $file.Mode 
 }
 if ($file.Mode -notlike "d*") {
 	continue
 }
 $myobj = "" | Select-Object Folder,ACL,level
 $myobj.Folder = $file
 $myobj.ACL = Get-Acl $file.FullName
 $myobj.level = $matches - 1
 $colACLs += $myobj
 }
 
 	'<div class="gposummary">' | Add-Content $report
 	
 	for ($i = 0; $i -lt $colACLs.count; $i++) {
 		drawDirectory ([ref] $colACLs[$i]) | Add-Content $report
 	}
 	'</div></body></html>' | Add-Content $report
 	
`

