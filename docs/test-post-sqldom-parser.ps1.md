---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6359
Published Date: 
Archived Date: 2016-05-28t02
---

#  - 

## Description

test post sqldom, parser

## Comments



## Usage



## TODO



## function

``

## Code

`#
 #
 function parsers {
 	param( $versions );
 	
 	#$sqldom = [System.Reflection.Assembly]::LoadFrom("C:\Program Files\Microsoft SQL Server\110\SDK\Assemblies\Microsoft.SqlServer.TransactSql.ScriptDom.dll");
 	$sqldom = [System.Reflection.Assembly]::LoadWithPartialName("Microsoft.SqlServer.TransactSql.ScriptDom");
 	if (-not $sqldom) {
 		write-host "[ERR ] Please install the SQLDOM.MSI from the SQL 2012 Feature Pack web page http://www.microsoft.com/en-us/download/details.aspx?id=35580" -foregroundcolor red
 		exit;
 	}
 	
 	$create_flgs = [ordered] @{};
 	if ($versions -eq $null) {
 		if ([bool] ($sqldom.gettypes() | where-object {$_.name -eq "TSql80Parser" } )) { $create_flgs.add("SQLServer2000", $true); }
 		if ([bool] ($sqldom.gettypes() | where-object {$_.name -eq "TSql90Parser" } )) { $create_flgs.add("SQLServer2005", $true); }
 		if ([bool] ($sqldom.gettypes() | where-object {$_.name -eq "TSql100Parser"} )) { $create_flgs.add("SQLServer2008", $true); }
 		if ([bool] ($sqldom.gettypes() | where-object {$_.name -eq "TSql110Parser"} )) { $create_flgs.add("SQLServer2012", $true); }
 		if ([bool] ($sqldom.gettypes() | where-object {$_.name -eq "TSql120Parser"} )) { $create_flgs.add("SQLServer2014", $true); }
 	} else {
 		foreach ($version in $versions) {
 			if ( (("SQLServer2000Sql80").contains($version) ) -and (-not $create_flgs.contains("SQLServer2000")) ) { $create_flgs.add("SQLServer2000", $true); }
 			if ( (("SQLServer2005Sql90").contains($version) ) -and (-not $create_flgs.contains("SQLServer2005")) ) { $create_flgs.add("SQLServer2005", $true); }
 			if ( (("SQLServer2008Sql100").contains($version)) -and (-not $create_flgs.contains("SQLServer2008")) ) { $create_flgs.add("SQLServer2008", $true); }
 			if ( (("SQLServer2012Sql110").contains($version)) -and (-not $create_flgs.contains("SQLServer2012")) ) { $create_flgs.add("SQLServer2012", $true); }
 			if ( (("SQLServer2014Sql120").contains($version)) -and (-not $create_flgs.contains("SQLServer2014")) ) { $create_flgs.add("SQLServer2014", $true); }
 		}
 	}
 
 	$parsers = [ordered] @{};
 	foreach ($version in $create_flgs.keys) {
 		if ($create_flgs[$version]) {
 			switch($version) {
 				"SQLServer2000" { $parsers.add($version, $(new-object Microsoft.SqlServer.TransactSql.ScriptDom.TSql80Parser($false)));  } 
 				"SQLServer2005" { $parsers.add($version, $(new-object Microsoft.SqlServer.TransactSql.ScriptDom.TSql90Parser($false)));  } 
 				"SQLServer2008" { $parsers.add($version, $(new-object Microsoft.SqlServer.TransactSql.ScriptDom.TSql100Parser($false))); } 
 				"SQLServer2012" { $parsers.add($version, $(new-object Microsoft.SqlServer.TransactSql.ScriptDom.TSql110Parser($false))); } 
 				"SQLServer2014" { $parsers.add($version, $(new-object Microsoft.SqlServer.TransactSql.ScriptDom.TSql120Parser($false))); } 
 			}
 		}
 	}
 	
 	return $parsers
 }
 
 function parser {
 	param( $version );
 	$parsers = parsers($version);
 	foreach ($parser in $parsers.keys) { 
 	}
 	return $parsers[$parser];
 }
 
 
 Export-ModuleMember -Function parsers
 Export-ModuleMember -Function parser
`

