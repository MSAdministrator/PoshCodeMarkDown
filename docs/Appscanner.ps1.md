---
Author: adam liquorish
Publisher: 
Copyright: 
Email: 
Version: 0.10
Encoding: ascii
License: cc0
PoshCode ID: 3102
Published Date: 2012-12-15t20
Archived Date: 2012-01-14t07
---

# appscanner - 

## Description

file access scanner for acl and applocker policy.  scan a particular file/folder for write/execute access for acl and applocker based on the rights of a supplied user.  can be used to test the success of an applocker policy.  or to simulate what access a particular user would have. scripts will also show direct membership and inherited membership for a user.  output is a table formatted in html.  the following user types can be used; domain,local and domain cached.  the domain cached can only be used when you are logged in as the cached user.  access denied exception errors may be produced when attempting to scan a folder that the particular user doesnt have access to(  errors are not fatal and will not effect the script outcome).

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 #######################
 
 
 
 
 
 
 
 
 #
 
 
 
 
 
 
 #######################
 
   
 
 
 param(
 
 [Parameter(Mandatory=$true,
 
     HelpMessage="Enter Path to be processed.")]
 
     [ValidateNotNullOrEmpty()]
 
     [string]$path,
 
 [Parameter(Mandatory=$true,
 
     HelpMessage="Enter User to be processed, as either builtin\<user> or <domain>\<user>.")]
 
     [ValidateNotNullOrEmpty()]
 
     [string]$user=$(Read-Host -prompt "User"),
 
 
 [Parameter(Mandatory=$true,
 
     HelpMessage="Enter Applocker XML to be utilised ie c:\applocker.xml, or type local to use effective policy for workstation")]
 
     [ValidateNotNullOrEmpty()]
 
     [string]$applockerpolicy=$(Read-Host -prompt "Path to applocker policy xml file, or type local to use effective policy for workstation"),
 
 [Parameter(Mandatory=$true,
 
     HelpMessage="Enter Path for ouput ie c:\Temp\output.html.")]
 
     [ValidateNotNullOrEmpty()]
 
     [string]$outputpath=$(Read-Host -prompt "Path for Output"),
 
 [Parameter(Mandatory=$true,
 
     HelpMessage="Is the user a Domain/Local/Cached User.[Domain,Local,Cached]")]
 
     [ValidateNotNullOrEmpty()]
 
     [ValidateSet("Domain","Local","Cached")]
 
     [string]$UserStatus=$(Read-Host -prompt "Is the user a Domain/Local/Cached User.[Domain,Local,Cached]"),
 
 [Parameter(Mandatory=$true,
 
     HelpMessage="Enter Log Directory for ouput ie c:\Temp\")]
 
     [ValidateNotNullOrEmpty()]
 
     [string]$logdirectory=$(Read-Host -prompt "Log Directory")
 
 )
 
 
 
 $logfilename="$(get-date -format yyyy-MM-dd-hh-mm-ss).txt" 
 
 $logfile=$logdirectory+$logfilename
 
 if ($host.name -match 'ise')
 
 {
 
     write-host "Warning: Running in Windows Powershell ISE, Transcript logging will not be running" -foregroundcolor red
 
     $null
 
 }
 
 else
 
 {
 
     write-host "Running in Powershell Console, Transcript logging will now start" -foregroundcolor blue
 
     try{
 
         start-transcript -path $logfile
 
     }
 
 
     catch [System.IO.DirectoryNotFoundException]{
 
         write-host "Critical: Parent Path to save $logfile not found." -foregroundcolor red
 
         read-host "Press enter to exit"
 
     }
 
 
     catch [System.Management.Automation.RuntimeException]{
 
         write-host "Critical: Write access to $logfile is denied unable to save log file." -foregroundcolor red
 
         read-host "Press enter key to exit"
 
     }
 
 }
 
 
 
 $ticksymbol=[char]10004
 
 $errorsymbol=[char]10008
 
 $asterisksymbol=[char]10033
 
 $dict=@{}
 
 $t=$null
 
 $hashtable=$null
 
 $u=$null
 
 $Pathvalid=test-path $path
 
 $Pathvalidpolicy=test-path $applockerpolicy
 
 $direct=$null
 
 $inherited=$null
 
 
 
 
 
 
 
 $a="<style>"
 
 $a=$a +"TABLE{border-width: 1px;border-style: solid;border-color: black;border-collapse: collapse;}"
 
 $a=$a +"TH{border-width: 1px;padding: 0px;border-style: solid;border-color: black;background-color:thistle}"
 
 $a=$a +"TD{border-width: 1px;padding: 0px;border-style: solid;border-color: black;}"
 
 $a=$a +"</style>"
 
 $header= "<h1>List of Processed Files</h1>"
 
 
 
 #$currentprincipal=new-object Security.Principal.WindowsPrincipal([Security.Principal.WindowsIdentity]::GetCurrent())
 
 #& {
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 try
 
 {
 
     if((get-wmiobject -cl win32_operatingsystem).version -gt "6")
 
     {
 
         write-host "$ticksymbol Win Vista or higher detected, Importing Applocker Module" -foregroundcolor blue
 
         if((get-module -listavailable|foreach-object {$_.name}) -contains "applocker")
 
         {
 
             import-module applocker
 
             write-host "Successfully imported applocker module" -foregroundcolor blue
 
         }
 
         else
 
         {
 
             write-host "Critical: Applocker module cannot be found try logging in as administrator" -foregroundcolor red
 
             read-host "Press enter to quit"
 
             exit
 
         }
 
     }
 
     else
 
     {
 
         "Critical: $errorsymbol Exiting....An operating system lower that Windows Vista has been detected.  Script can only be run on Vista or higher."
 
         read-host "Press Enter key to exit"
 
         Exit
 
     }
 
 }
 
 catch
 
 {
 
     write-host "Critical: Error encountered loading applocker module" -foregroundcolor red
 
     read-host "Press Enter key to exit"
 
     Exit
 
 }
 
 
 
 
 
 
 
 
 
 
 
 
 
 if ($Pathvalid -eq "True")
 
 
     {
 
     if ($applockerpolicy -eq "local")
 
 
     {
 
 
         if((get-applockerpolicy -effective -xml ) -like "*Rule*")
 
         {
 
             write-host "$ticksymbol A valid Applocker Policy is currently applied to this workstation" -foregroundcolor blue
 
             write-host "Warning: A path is required to save local applied applocker policy for usage" -foregroundcolor red
 
             $applockerpolicy=read-host "Enter path, ie c:\temp\applockerpolicy.xml" 
 
             write-host "$asterisksymbol Effective applied Applocker Policy for this workstation has been selected, policy will be output to $applockerpolicy" -foregroundcolor blue
 
 
             try{
 
                 get-applockerpolicy -effective -xml >$applockerpolicy
 
             }
 
 
             catch [System.IO.DirectoryNotFoundException]{
 
                 write-host "Critical: Parent Path to save $applockerpolicy not found." -foregroundcolor red
 
                 read-host "Press enter to exit"
 
             }
 
 
             catch [System.Management.Automation.RuntimeException]{
 
                 write-host "Critical: Write access to $applockerpolicy is denied unable to export policy." -foregroundcolor red
 
                 read-host "Press enter key to exit"
 
             }
 
         }
 
         else
 
         {
 
             write-host "Critical: $errorsymbol Exiting....An applocker policy has not been applied to this workstation" -foregroundcolor red
 
             read-host "Press Enter key to exit"
 
             exit
 
         }
 
         
 
     }
 
     elseif ($Pathvalidpolicy -eq "True")
 
     {
 
          write-host "$ticksymbol Valid XML file supplied for Applocker Policy" -foregroundcolor blue
 
     }
 
     else
 
     {
 
         write-host "Critical: $errorsymbol Exiting....Invalid path for applocker policy xml file, File Doesn't exist!" -foreground red
 
         read-host "Press Enter key to exit"
 
         exit
 
     }  
 
 
     $starttime=get-date
 
     "Stage 1 of 7, Enumerating Groups User is a member of, including inherited groups"
 
 
     add-type -AssemblyName System.DirectoryServices.AccountManagement
 
     $domain = (Get-wmiobject Win32_ComputerSystem).Domain
 
     $ping=new-object system.net.networkinformation.ping
 
     
 
 
     function groupfind  
 
     {
 
 
         $principal = new-object System.DirectoryServices.AccountManagement.PrincipalContext $ctype,$domain
 
         $idtype = [System.DirectoryServices.AccountManagement.IdentityType]::SamAccountName
 
         $groupPrincipal = [System.DirectoryServices.AccountManagement.UserPrincipal]::FindByIdentity($principal, $idtype, $user)
 
 
 
         set-variable -name groupout -value $groupprincipal.GetAuthorizationGroups() -scope global
 
     }
 
 
     
 
 
     If($userstatus -eq "Domain")
 
     {
 
         try {
 
             $domainName = [System.DirectoryServices.ActiveDirectory.Domain]::GetComputerDomain() | select -ExpandProperty Name
 
             $isDomain = $domainName -match $domain
 
             $domain =$domainname
 
             write-host "Workstation is part of a domain" -foregroundcolor blue
 
 
             if ($ping.send(([System.DirectoryServices.ActiveDirectory.Domain]::GetComputerDomain()).pdcroleowner.name).status -eq "Success")
 
             {
 
                 $ctype = [System.DirectoryServices.AccountManagement.ContextType]::Domain
 
                 write-host "Successfully contacted Domain controller, using Domain account information." -foregroundcolor blue
 
 
                 groupfind
 
             }
 
             else
 
             {
 
                 write-host "Critical: Domain Controller not contactable!" -foregroundcolor red
 
                 read-host "Press Enter key to exit"
 
                 exit
 
             }
 
         }
 
         catch {
 
             write-host "Critical: Computer is not part of a domain" -foregroundcolor red
 
             read-host "Press Enter key to exit"
 
             exit
 
         }
 
     }
 
     elseif($userstatus -eq "Local")
 
     {
 
 
     $computername="$env:computername"
 
     $computer=[ADSI]"WinNT://$computername,computer"
 
     $localuserlist=$computer.psbase.children|where-object {$_.psbase.schemaclassname -eq 'user'}
 
     $localuserlistfilt=foreach($useritem in $localuserlist){$useritem.name}
 
 
         if($localuserlistfilt -contains $user)
 
         {
 
             write-host "Verified user is a part of local SAM database" -foregroundcolor blue
 
             $domain=(Get-wmiobject Win32_ComputerSystem).Name
 
             $ctype = [System.DirectoryServices.AccountManagement.ContextType]::Machine
 
 
             groupfind
 
         }
 
         else
 
         {
 
             write-host "Critical: User is not a local user" -foregroundcolor red
 
             read-host "Press Enter key to exit"
 
             exit
 
         }
 
     }
 
     elseif($userstatus -eq "Cached")
 
     {
 
         try {
 
 
         $computername="$env:computername"
 
         $computer=[ADSI]"WinNT://$computername,computer"
 
         $localuserlist=$computer.psbase.children|where-object {$_.psbase.schemaclassname -eq 'user'}
 
         $localuserlistfilt=foreach($useritem in $localuserlist){$useritem.name}
 
 
             if($localuserlistfilt -contains $user)
 
             {
 
                 write-host "Critical: User is a part of local SAM database, therefore user is not cached." -foregroundcolor red
 
                 read-host "Press Enter key to exit"
 
                 exit
 
             }
 
             else
 
             {
 
 
                if((gwmi win32_computersystem).username -like "*$user")
 
                 {
 
                     write-host "Verified user is a cached user" -foregroundcolor blue
 
                     $groupout=[system.security.principal.windowsidentity]::getcurrent().groups|foreach-object {$_.translate([system.security.principal.ntaccount])}
 
                 }
 
                 else
 
                 {
 
                     write-host "Critical: Logged on user doesn't match queried user, therefore User is not a cached user" -foregroundcolor red
 
                     read-host "Press Enter key to exit"
 
                     exit
 
                 }
 
             }                    
 
         }
 
         catch {
 
             write-host "Critical: User is not cached" -foregroundcolor red
 
             read-host "Press Enter key to exit"
 
             exit
 
         }
 
     }
 
     else
 
     {
 
         write-host "Critical: Please use Local,Domain or Cached" -foregroundcolor red
 
         read-host "Press Enter key to exit"
 
         exit
 
     }
 
     
 
 
 
     
 
     "Stage 1 of 7, Finished Scanning Group Membership"
 
     "Stage 1 of 7, Outputting Group Membership hierarchy"
 
 
     $groupfilter=@($user)
 
 
     $groupfilter+=foreach($groupname in $groupout){$groupname.name}
 
 
     $domaincut=$domain -match "\w+[A-Za-z0-9-]+"
 
     $domaincutvalue=$matches.values
 
     $query="ASSOCIATORS OF {Win32_Account.Name='$user',Domain='$domaincutvalue'} WHERE ResultRole=GroupComponent ResultClass=Win32_Account"
 
     $directmembership=get-wmiobject -query $query
 
     $directmembershipresults=foreach($directmember in $directmembership){$directmember.name}
 
     $directmembershipresultsfiltered=$directmembershipresults|select-object -unique
 
     
 
     "#####################################################"
 
 
 
     foreach ($group in $groupfilter){
 
         if($directmembershipresultsfiltered -contains $group){
 
             $direct+=@($group)}
 
         elseif($group -eq $user){
 
             $null}
 
         else{$inherited+=@($group)}
 
       }
 
 
     write-host "-$user" -foregroundcolor darkgreen
 
 
     foreach($member in $direct){
 
         write-host "->$member" -foregroundcolor red
 
         }
 
     foreach($member in $inherited){
 
         write-host "-->$member" -foregroundcolor blue
 
         }
 
     "#####################################################"
 
     "Stage 1 of 7 Complete"
 
 
     
 
 
     $count=0
 
     "Stage 2 of 7 $path is populating a variable "
 
     Get-Childitem $path -recurse -outvariable objects|where-object{write-progress "Stage 2 of 7 Recursing items to variable, Examining $($_.fullname)...." "Found  $count items";"$($_.fullname)"}|foreach-object {$count++}
 
     "Stage 2 of 7 $path has been populated into a variable"
 
 
     
 
 
     "Stage 3 of 7 Processing ACL on files to index"
 
      $max=$objects.length
 
 
      $filteracl ={$groupfilter -like $_.IdentityReference.value.split("\")[1] -and ($_.FileSystemRights -band 131241 -or $_.FileSystemRights -band 278)}
 
 
      foreach ($i in $objects)
 
        {
 
             $dict[$i.fullname]=@{user="";Permission=""} 
 
             $t++
 
             $i.GetAccessControl().Access |Where $filteracl|foreach {$dict.($i.Fullname).User+=($_.IdentityReference,",");$dict.($i.Fullname).Permission=$_.FileSystemRights} 
 
             Write-Progress -activity "Stage 3 of 7 Processing File Permissions to index" -status "$t of $max" -PercentComplete (($t/$objects.count)*100) -CurrentOperation $i.fullname 
 
        }
 
     
 
     "Stage 3 of 7 Complete"
 
 
     
 
 
     "Stage 4 of 7 Removing duplicate identities"
 
 
     $t=$null
 
 
       foreach ($i in $objects)
 
     {
 
         $t++
 
         $identarray=$dict[$i.fullname].user;$dict[$i.fullname].user=$null;$splitidentarray=$identarray -split ",";$uniqueidentarray=$splitidentarray|sort-object -unique;$uniqueidentarray -join ","|foreach {$dict.($i.fullname).User+=($_)}
 
         Write-Progress -activity "Stage 4 of 7 Removing Username/Group Duplicates" -status "$t of $max" -PercentComplete (($t/$objects.count)*100) -CurrentOperation $i.fullname 
 
     }
 
     "Stage 4 of 7 Complete"
 
 
 
 
 
     "Stage 5 of 7 Processing Applocker policy on files"
 
 
     $Applockerfileextlist=".bat",".cmd",".dll",".exe",".js",".msi",".msp","ocx",".psq",".vbs"
 
     $userpol=$objects|where {$Applockerfileextlist -contains $_.Extension}|convert-path|test-applockerpolicy $applockerpolicy -User $user
 
     $userobjpol = $userpol|select-object PolicyDecision,FilePath,MatchingRule
 
     $userobjpolcount=0
 
     $userobjpol|foreach {
 
         $userobjpolcount++
 
         $dict[$_.FilePath] += @{ PolicyDecision = $_.PolicyDecision;MatchingRule= $_.MatchingRule}
 
         Write-progress -activity "Stage 5 of 7 Processing AppLockers results:" -status "$userobjpolcount of $($userobjpol.count)" -PercentComplete (($userobjpolcount/$userobjpol.count)*100) -CurrentOperation $_
 
     }
 
     "Stage 5 of 7 Complete"
 
 
 
 
 
     "Stage 6 0f 7 Preparing format of results for html Report"
 
     $max2=$dict.count
 
     $hashtable=foreach($j in $dict.keys){
 
         $u++
 
         New-Object -TypeName PSObject -Property @{Path=$j
 
         User=$dict.$j.user
 
         Permission=$dict.$j.Permission
 
         MatchingRule=$dict.$j.MatchingRule
 
         PolicyDecision=$dict.$j.PolicyDecision
 
     }
 
     Write-Progress -activity "Stage 6 of 7 Processing Dictionary to properties" -status "$u of $max2" -PercentComplete (($u/$max2)*100) -CurrentOperation $_}
 
     "Stage 6 of 7 Complete, $u files scanned of $max for Applocker scan"
 
 
 
 
 
     "Stage 7 of 7 Outputting to file $outputpath"
 
     try{
 
         $hashtable|sort-object Path|ConvertTo-Html -head $header -title "ACL List" -body $a|Set-Content $outputpath
 
     }
 
 
     catch [System.IO.DirectoryNotFoundException]{
 
         write-host "Critical: Parent Path to save $outputpath not found." -foregroundcolor red
 
         read-host "Press enter to exit"
 
     }
 
 
     catch [System.Management.Automation.RuntimeException]{
 
         write-host "Critical: Write access to $outputpath is denied unable to export results." -foregroundcolor red
 
         read-host "Press enter key to exit"
 
     }
 
 
     $endtime=get-date
 
     $totaltime=$endtime-$starttime
 
     $totaltimehours=$totaltime.hours
 
     $totaltimeminutes=$totaltime.minutes
 
     $outputsize=get-item "$outputpath"|foreach {echo($_.length/1mb).tostring("0.00 MB")}
 
     "Stage 7 of 7, Scanned $max files of $path is complete in $totaltimehours hours and $totaltimeminutes minutes, $outputpath is $outputsize."
 
 
 
 
     if ($host.name -match 'ise')
 
     {
 
         $null
 
     }
 
     else
 
     {
 
         "Running Log output to $logfile"
 
         stop-transcript >$null
 
     }
 
 
     $delete=read-host -prompt "Would you like the Applocker policy file $applockerpolicy deleted,YES/NO"
 
     if($delete -eq "yes")
 
     {
 
         del $applockerpolicy
 
     }
 
     else
 
     {
 
         write-host "Warning: $asterisksymbol You chose not to delete file $applockerpolicy, Application will now exit....." -foregroundcolor red
 
         read-host "Press Enter key to exit"
 
     }
 
 }
 
     
 
 
 else
 
 {
 
     write-host "Critical: $errorsymbol Exiting...Invalid path supplied for processing" -foregroundcolor red
 
     read-host "Press Enter key to exit"
 
     exit
 
 }
`

