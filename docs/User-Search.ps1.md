---
Author: steelestengergm
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4303
Published Date: 2016-07-16t18
Archived Date: 2016-05-30t07
---

# user search - 

## Description

author

## Comments



## Usage



## TODO



## module

`find-user`

## Code

`#
 #
 import-module showui
 Add-PSSnapin Quest.ActiveRoles.ADManagement
 
 function Find-User {
 param ($First, $Last, $Address)
         $domains = @{
             Fabrikam="dc=fabrikam,dc=com"
             AdventureWorks="dc=adventure-works,dc=com"
         }
     [Array] $found = @() 
 
             $First = $First.trim()
             $Last = $Last.trim()
             $Address = $Address.trim()
 
     if ($Address.length -gt 0) {
         if ($Address.length -gt 0) {
             foreach ($domain in $domains.values) {
                 $found += @(get-qaduser -Email $Address -IncludeAllProperties -SearchRoot $domain | select Domain, Firstname, LastName, samaccountname, email, AccountIsDisabled, LastLogon
 ) 
             }
         }
     }
 
     if (($First.length -gt 0) -and ($Last.length -gt 0)) {
         if (($First -ne "") -and ($Last -ne "")) {
             $fullname = "$First $Last"
 
             foreach ($domain in $domains.values) {
                 $found += @(get-qaduser -Displayname $fullname -IncludeAllProperties -SearchRoot $domain | Select Domain, Firstname, LastName, samaccountname, email, AccountIsDisabled, LastLogon) 
                 $found.count | out-null
             }
         }
     }
 
     elseif ($First -ne "") {
         if ($First.length -gt 0) {
             foreach ($domain in $domains.values) {
                 $found += @(get-qaduser -FirstName $First -IncludeAllProperties -SearchRoot $domain | select Domain, Firstname, LastName, samaccountname, email, AccountIsDisabled, LastLogon)
             }
         }
     }
 
     elseif ($Last -ne "") {
         if ($Last.length -gt 0) {
             foreach ($domain in $domains.values) {
                 $found += @(get-qaduser -LastName $Last -IncludeAllProperties -SearchRoot $domain | Select Domain, Firstname, LastName, samaccountname, email, AccountIsDisabled, LastLogon)
                 }
             }
         }
 
         $none = $null
         if ($found.count -eq 0) {
             $none = "Your search did not return any results"
             return $none 
         }
         else {
         return $found
         }
 }
 
 
 function Find-MultiUser {
 param ($multi)
     [Array] $found = @()
 
     $domains = @{
             Fabrikam="dc=fabrikam,dc=com"
             AdventureWorks="dc=adventure-works,dc=com"
         }
         $users = @($multi.split("`n"))
 
         $user = $null
 
         foreach ($user in $users) {
             $user = $user.trim()
             write-host "input: " `t $user -f "Green" | out-null
 
             if (($user.length -gt 0) -and ($user -ne "")) {
 
                 if ($user.contains('@')) {
                     write-host "Multi Email" -f "Green" | out-null
                     
                     foreach ($domain in $domains.values){
                         $found = $found + @(get-qaduser -Email $user -IncludeAllProperties -SearchRoot $domain | Select Domain, Firstname, LastName, samaccountname, email, AccountIsDisabled, LastLogon)
                     }
                 }
 
                 elseif($user.contains(" ")) {
                         write-host "Search Fullname" | out-null
                         foreach ($domain in $domains.values){
                         write-host $domain
                             $found = $found + @(get-qaduser -Displayname $user -IncludeAllProperties -SearchRoot $domain | Select Domain, Firstname, LastName, samaccountname, email, AccountIsDisabled, LastLogon)
                         }
                     }
                 
                 else {
                     write-host "Search Lastname" | out-null
                     foreach ($domain in $domains.values){
                         $found = $found + @(get-qaduser -LastName $user -IncludeAllProperties -SearchRoot $domain | Select Domain, Firstname, LastName, samaccountname, email, AccountIsDisabled, LastLogon)
                     }
 
                     $found.Length | out-null
                     write-host "Search FirstName" | out-null
                         foreach ($domain in $domains.values) {
                         $found = $found + @(get-qaduser -FirstName $user -IncludeAllProperties -SearchRoot $domain | Select Domain, Firstname, LastName, samaccountname, email, AccountIsDisabled, LastLogon)
                         }
                     
 
         $none = $null
         if ($found.length -eq 0) {
             $none = "Your search did not return any results"
             return $none 
         }
         else {
         return $found
         }
     }
     
 
 New-Window -Title "User Search" -SizeToContent WidthAndHeight -WindowStartupLocation CenterScreen -ResizeMode NoResize -Background "black" {
 New-StackPanel -name Stack -ControlName 'FindUser' -Orientation Vertical -Background "White" -HorizontalAlignment Left -margin 10 {
         RadioButton "_Single User" -Name UserType -margin 10 -On_Checked {
             $Button_Multi.IsEnabled = $false;
             $Button_Single.IsEnabled = $true;
             $FirstName.ISEnabled = $true;
             $CheckFirst.ISEnabled = $true;
             $CheckFirst.IsChecked = $true;
             $FirstName.text = "";
             $Lastname.ISEnabled = $true;
             $EmailAddress.ISEnabled = $false;
             $CheckLast.ISEnabled = $true;
             $CheckLast.IsChecked = $true;
             $LastName.text = "";
             $EmailAddress.IsEnabled = $false;
             $CheckEmail.ISEnabled = $true;
             $CheckEmail.ISChecked = $false;
             $EmailAddress.text = "";
             $Multi.IsEnabled = $false;
             $multi.text = ""
             }
         RadioButton "_Multi User" -Name UserType -margin 10 -On_Checked {
             $Button_Single.IsEnabled = $false;
             $Button_Multi.IsEnabled = $true;
             $Multi.IsEnabled = $true;
             $Multi.text = "";
             $CheckFirst.ISEnabled = $false
             $CheckLast.ISEnabled = $false
             $FirstName.ISEnabled = $false;
             $CheckFirst.ISChecked = $false;
             $FirstName.text = "";
             $Lastname.ISEnabled = $false;
             $CheckLast.ISChecked = $false;
             $LastName.text = ""  
             $EmailAddress.ISEnabled = $false;
             $CheckEmail.ISEnabled = $false;
             $CheckEmail.ISChecked = $false;
             $EmailAddress.text = ""
             }
 New-Grid -Name UserInfo -Isenabled:$true -HorizontalAlignment Left -Rows 7 -Columns 3 -background "white" -margin 25 {
             New-CheckBox -name CheckFirst -VerticalAlignment Center -Isenabled:$false -IsChecked:$False -Row 0 -Column 0 -On_Checked {
                 $EmailAddress.IsEnabled = $false;
                 $CheckEmail.IsChecked = $false;
                 $FirstName.ISEnabled = $true} -On_Unchecked {$FirstName.text = $null; $FirstName.ISEnabled = $false}
             label "First Name" -Row 0 -column 1 -HorizontalAlignment Left
             TextBox -Name FirstName -IsEnabled:$False -Row 0 -Column 2 -MinWidth 300
             #
             New-CheckBox -name CheckLast -VerticalAlignment Center -Isenabled:$false -IsChecked:$False -Row 1 -Column 0 -On_Checked {
                 $EmailAddress.IsEnabled = $false;
                 $CheckEmail.IsChecked = $false;
                 $LastName.ISEnabled = $true} -On_Unchecked {$LastName.text = $null; $LastName.ISEnabled = $false}
             label "Last Name" -Row 1 -column 1 -HorizontalAlignment Left
             TextBox -Name Lastname -IsEnabled:$False -Row 1 -Column 2 -MinWidth 300
             #
             New-CheckBox -name CheckEmail -VerticalAlignment Center -Isenabled:$false -IsChecked:$False -Row 2 -Column 0 -On_Checked {
                 $CheckFirst.ISEnabled = $true;
                 $CheckFirst.ISChecked = $false;
                 $CheckLast.ISEnabled = $true;
                 $CheckLast.ISChecked = $false;
                 $EmailAddress.IsEnabled = $true
                 } -On_Unchecked {$EmailAddress.text = $null; $EmailAddress.ISEnabled = $false}
             label "Email Address" -Row 2 -column 1 -HorizontalAlignment Left
 
             TextBox -Name EmailAddress -IsEnabled:$False -Row 2 -Column 2 -MinWidth 300
             new-button -name Button_Single "Search" -IsEnabled:$false -Row 3 -Column 1 -width 100 -HorizontalAlignment Left -VerticalAlignment bottom -Margin 20 -On_Click {
                 cls;
                 $window.Cursor = "Wait";
                 Find-User $FirstName.text $LastName.text $EmailAddress.text "Single" | Out-GridView -Title "Single User Results - Search" -Passthru | foreach {
                     $export = $null
                     $export = [Environment]::GetFolderPath("Desktop")
                     Export-Csv -InputObject $_ -Delimiter (",") -Encoding ASCII -LiteralPath "$export\search_results.csv" -NoClobber -Append -Force
                 }
                 $window.Cursor = ""
             }
             label " " -Row 4 -Column 0
             label "Multi User" -Row 5 -Column 1 -HorizontalAlignment Left
             TextBox -Row 5 -Column 2 -MinLines 20 -Name Multi -IsEnabled:$False -AcceptsReturn:$true -TextWrapping Wrap -VerticalScrollBarVisibility Auto
             new-button -name Button_Multi "Search" -IsEnabled:$False -Row 6 -Column 1 -width 100 -HorizontalAlignment Left -VerticalAlignment bottom -Margin 20 -On_Click {
                 cls;
                 $window.Cursor = "Wait";
                 Find-MultiUser $multi.text | Out-GridView -Title "Mutli User Results - Search" -Passthru | foreach {
                     $export = $null
                     $export = [Environment]::GetFolderPath("Desktop")
                     Export-Csv -InputObject $_ -Delimiter (",") -Encoding ASCII -Force -LiteralPath "$export\search_results.csv" -NoClobber -Append -Force
                 }
                 $window.Cursor = ""
             }
         }
 
 New-Grid -Name Bottom -IsEnabled:$True -HorizontalAlignment Left -Rows 1 -Columns 3 -margin 15 {
         
         new-button -Name Button_Quit "Quit" -IsEnabled:$true -width 100 -HorizontalAlignment Left -Row 3 -Column 2 -on_click {
         Close-Control
         }
     }
 }
 }-show
`

