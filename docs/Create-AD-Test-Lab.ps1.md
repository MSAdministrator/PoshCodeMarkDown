---
Author: ted wagner
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1666
Published Date: 2011-02-26t06
Archived Date: 2015-01-23t12
---

# create ad test lab - 

## Description

this script expands on dmitry sotnikov’s blog post on creating demo active directory environments.  http

## Comments



## Usage



## TODO



## script

`create-labous`

## Code

`#
 #
 ###
 ###
 ###
 ###
 ###
 ###
 ###
 ###
 ###
 ###
 ###
 ##################################################
 ##################################################
 ###
 
 . $ScriptLib\functions.ps1
 
 function create-LabOUs (){
 	for ($i = 0; $i -le $DeptOUs.Length - 1; $i++){
 		$OUName = "Test Lab Container - " + $DeptOUs[$i]
 		$CreateDeptOU += @(new-QADObject -ParentContainer $RootDomain.RootDomainNamingContext `
 		-type 'organizationalUnit' -NamingProperty 'ou' -name $DeptOUs[$i] -description $OUName )
 	}
 
 	foreach ($DeptOU in $CreateDeptOU){
 		for ($i = 0; $i -le $ChildOUs.Length - 1; $i++){
 			new-qadObject -ParentContainer $DeptOU.DN -type 'organizationalUnit' -NamingProperty 'ou' `
 			-name $ChildOUs[$i]
 		}
 	}
 }
 
 function New-RandomADUser (){
 	$rnd = New-Object System.Random
 
 	if($rnd.next(2) -eq 1) {
 		$fn = $firstm[$rnd.next($firstm.length)]
 	} else {
 		$fn = $firstf[$rnd.next($firstf.length)]
 	}
 	$ln = $last[$rnd.next($last.length)]
 
 	$ln = $ln[0] + $ln.substring(1, $ln.length - 1).ToLower()
 	$fn = $fn[0] + $fn.substring(1, $fn.length - 1).ToLower()
 
 	$city = $cities[$rnd.next($cities.length)]
 	$dept = $depts[$rnd.next($depts.length)]
 
 	$SName = ($fn.substring(0,1) + $ln)
 
 	switch ($dept){
 		$DeptContainers[0].name {$UserOU = Get-QADObject -SearchRoot $DeptContainers[0].DN | `
 			where { $_.DN -match "Users" -and $_.Type -ne "user" }} 
 		$DeptContainers[1].name {$UserOU = Get-QADObject -SearchRoot $DeptContainers[1].DN | `
 			where { $_.DN -match "Users" -and $_.Type -ne "user" }}
 		$DeptContainers[2].name {$UserOU = Get-QADObject -SearchRoot $DeptContainers[2].DN | `
 			where { $_.DN -match "Users" -and $_.Type -ne "user" }}
 		$DeptContainers[3].name {$UserOU = Get-QADObject -SearchRoot $DeptContainers[3].DN | `
 			where { $_.DN -match "Users" -and $_.Type -ne "user" }}
 		$DeptContainers[4].name {$UserOU = Get-QADObject -SearchRoot $DeptContainers[4].DN | `
 			where { $_.DN -match "Users" -and $_.Type -ne "user" }}
 		$DeptContainers[5].name {$UserOU = Get-QADObject -SearchRoot $DeptContainers[5].DN | `
 			where { $_.DN -match "Users" -and $_.Type -ne "user" }}
 		$DeptContainers[6].name {$UserOU = Get-QADObject -SearchRoot $DeptContainers[6].DN | `
 			where { $_.DN -match "Users" -and $_.Type -ne "user" }}
 		$DeptContainers[7].name {$UserOU = Get-QADObject -SearchRoot $DeptContainers[7].DN | `
 			where { $_.DN -match "Users" -and $_.Type -ne "user" }}
 		$DeptContainers[8].name {$UserOU = Get-QADObject -SearchRoot $DeptContainers[8].DN | `
 			where { $_.DN -match "Users" -and $_.Type -ne "user" }}
 		$DeptContainers[9].name {$UserOU = Get-QADObject -SearchRoot $DeptContainers[9].DN | `
 			where { $_.DN -match "Users" -and $_.Type -ne "user" }}
 	}
 
 	if ((get-qaduser $SName) -eq $null){
 		New-QADUser -Name "$ln`, $fn" -SamAccountName $SName -ParentContainer $UserOU -City $city `
 		-Department $dept -UserPassword "p@ssw0rd" -FirstName $fn -LastName $ln -DisplayName "$ln`, $fn" `
 		-Description "$city $dept" -Office $city | Enable-QADUser
 	}
 
 	switch ($dept){
 		$DeptContainers[0].name {$GroupOU = Get-QADObject -SearchRoot $DeptContainers[0].DN | `
 			where { $_.DN -match "Groups" -and $_.Type -ne "group" }} 
 		$DeptContainers[1].name {$GroupOU = Get-QADObject -SearchRoot $DeptContainers[1].DN | `
 			where { $_.DN -match "Groups" -and $_.Type -ne "group" }}
 		$DeptContainers[2].name {$GroupOU = Get-QADObject -SearchRoot $DeptContainers[2].DN | `
 			where { $_.DN -match "Groups" -and $_.Type -ne "group" }}
 		$DeptContainers[3].name {$GroupOU = Get-QADObject -SearchRoot $DeptContainers[3].DN | `
 			where { $_.DN -match "Groups" -and $_.Type -ne "group" }}
 		$DeptContainers[4].name {$GroupOU = Get-QADObject -SearchRoot $DeptContainers[4].DN | `
 			where { $_.DN -match "Groups" -and $_.Type -ne "group" }}
 		$DeptContainers[5].name {$GroupOU = Get-QADObject -SearchRoot $DeptContainers[5].DN | `
 			where { $_.DN -match "Groups" -and $_.Type -ne "group" }}
 		$DeptContainers[6].name {$GroupOU = Get-QADObject -SearchRoot $DeptContainers[6].DN | `
 			where { $_.DN -match "Groups" -and $_.Type -ne "group" }}
 		$DeptContainers[7].name {$GroupOU = Get-QADObject -SearchRoot $DeptContainers[7].DN | `
 			where { $_.DN -match "Groups" -and $_.Type -ne "group" }}
 		$DeptContainers[8].name {$GroupOU = Get-QADObject -SearchRoot $DeptContainers[8].DN | `
 			where { $_.DN -match "Groups" -and $_.Type -ne "group" }}
 		$DeptContainers[9].name {$GroupOU = Get-QADObject -SearchRoot $DeptContainers[9].DN | `
 			where { $_.DN -match "Groups" -and $_.Type -ne "group" }}
 	}
 
 	if ((get-QADGroup $dept) -eq $null){
 		New-QADGroup -Name $dept -SamAccountName $dept -ParentContainer $GroupOU -Description "$dept Users"
 	}
 
 	Get-QADUser $SName -SearchRoot $UserOU | Add-QADGroupMember -Identity { $_.Department }
 	
 	switch ($dept){
 		$DeptContainers[0].name {$ComputerOU = Get-QADObject -SearchRoot $DeptContainers[0].DN | `
 			where { $_.DN -match "Computers" -and $_.Type -ne "computer" }} 
 		$DeptContainers[1].name {$ComputerOU = Get-QADObject -SearchRoot $DeptContainers[1].DN | `
 			where { $_.DN -match "Computers" -and $_.Type -ne "computer" }}
 		$DeptContainers[2].name {$ComputerOU = Get-QADObject -SearchRoot $DeptContainers[2].DN | `
 			where { $_.DN -match "Computers" -and $_.Type -ne "computer" }}
 		$DeptContainers[3].name {$ComputerOU = Get-QADObject -SearchRoot $DeptContainers[3].DN | `
 			where { $_.DN -match "Computers" -and $_.Type -ne "computer" }}
 		$DeptContainers[4].name {$ComputerOU = Get-QADObject -SearchRoot $DeptContainers[4].DN | `
 			where { $_.DN -match "Computers" -and $_.Type -ne "computer" }}
 		$DeptContainers[5].name {$ComputerOU = Get-QADObject -SearchRoot $DeptContainers[5].DN | `
 			where { $_.DN -match "Computers" -and $_.Type -ne "computer" }}
 		$DeptContainers[6].name {$ComputerOU = Get-QADObject -SearchRoot $DeptContainers[6].DN | `
 			where { $_.DN -match "Computers" -and $_.Type -ne "computer" }}
 		$DeptContainers[7].name {$ComputerOU = Get-QADObject -SearchRoot $DeptContainers[7].DN | `
 			where { $_.DN -match "Computers" -and $_.Type -ne "computer" }}
 		$DeptContainers[8].name {$ComputerOU = Get-QADObject -SearchRoot $DeptContainers[8].DN | `
 			where { $_.DN -match "Computers" -and $_.Type -ne "computer" }}
 		$DeptContainers[9].name {$ComputerOU = Get-QADObject -SearchRoot $DeptContainers[9].DN | `
 			where { $_.DN -match "Computers" -and $_.Type -ne "computer" }}
 	}
 
 	if ((get-qadcomputer "$SName-Computer") -eq $null){
 		New-QADComputer -Name "$SName-Computer" -SamAccountName "$SName-Computer" -ParentContainer `
 		$ComputerOU -Location "$city $dept"
 	}
 }
 
 $TestQADSnapin = get-pssnapin | where { $_.Name -eq "Quest.ActiveRoles.ADManagement"} 
 if($TestQADSnapin -eq $null){
 	add-pssnapin -Name Quest.ActiveRoles.ADManagement -ErrorAction SilentlyContinue
 } 
 
 $num = 50
 
 $RootDomain = Get-QADRootDSE
 
 $DeptOUs = @(Get-Content "$ScriptLib\LabSetup\Departments.txt")
 $ChildOUs = @(Get-Content "$ScriptLib\labsetup\ous.txt")
 $cities = Get-Content C:\scripts\LabSetup\Cities.txt
 $depts = Get-Content C:\scripts\LabSetup\Departments.txt
 
 1..$num | ForEach-Object {
 	$last += @(Get-Content C:\scripts\LabSetup\surnames.txt | select-random)
 	$firstm += @(Get-Content C:\scripts\LabSetup\mgivennames.txt | select-random)
 	$firstf += @(Get-Content C:\scripts\LabSetup\fgivennames.txt | select-random)
 }
 
 
 create-LabOUs
 
 $DeptContainers = @(Get-QADObject -Type "organizationalUnit" | where {$_.Name -ne "Computers" -and $_.Name `
 	-ne "Groups" -and $_.Name -ne "Users" -and $_.Description -match "Test Lab Container"})
 
 foreach ($item in $DeptContainers){
 	$item.description
 }
 1..$num | ForEach-Object { New-RandomADUser }
 
 trap{
 	Write-Host "ERROR: script execution was terminated.`n" $_.Exception.Message
 	break
 }
`

