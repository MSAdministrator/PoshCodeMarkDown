---
Author: alphasun
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 6297
Published Date: 2016-04-08t21
Archived Date: 2016-12-03t15
---

# remote uninstall - 

## Description

this script allows you to perform application uninstalls remotely. the script prompts for an input file that contains target pcs (one on each line) and a search terms box.

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 ##########################################################################################
 ##########################################################################################
 ##########################################################################################
 ##########################################################################################
 
 
 ################################################
 ################################################
 
 Function ListPrograms {
 
 [cmdletbinding()]            
 
 [cmdletbinding()]
 param(
  [parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
  [string[]]$ComputerName = $env:computername            
 
 )            
 
 begin {
  $UninstallRegKey="SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall"
 }            
 
 process {
  foreach($Computer in $ComputerName) {
   Write-Verbose "Working on $Computer"
   if(Test-Connection -ComputerName $Computer -Count 1 -ea 0) {
    $HKLM   = [microsoft.win32.registrykey]::OpenRemoteBaseKey('LocalMachine',$computer)
    $UninstallRef  = $HKLM.OpenSubKey($UninstallRegKey)
    $Applications = $UninstallRef.GetSubKeyNames()            
 
    foreach ($App in $Applications) {
     $AppRegistryKey  = $UninstallRegKey + "\\" + $App
     $AppDetails   = $HKLM.OpenSubKey($AppRegistryKey)
     $AppGUID   = $App
     $AppDisplayName  = $($AppDetails.GetValue("DisplayName"))
     $AppVersion   = $($AppDetails.GetValue("DisplayVersion"))
     $AppPublisher  = $($AppDetails.GetValue("Publisher"))
     $AppInstalledDate = $($AppDetails.GetValue("InstallDate"))
     $AppUninstall  = $($AppDetails.GetValue("UninstallString"))
     if(!$AppDisplayName) { continue }
     $OutputObj = New-Object -TypeName PSobject
     $OutputObj | Add-Member -MemberType NoteProperty -Name ComputerName -Value $Computer.ToUpper()
     $OutputObj | Add-Member -MemberType NoteProperty -Name AppName -Value $AppDisplayName
     $OutputObj | Add-Member -MemberType NoteProperty -Name AppVersion -Value $AppVersion
     $OutputObj | Add-Member -MemberType NoteProperty -Name AppVendor -Value $AppPublisher
     $OutputObj | Add-Member -MemberType NoteProperty -Name InstalledDate -Value $AppInstalledDate
     $OutputObj | Add-Member -MemberType NoteProperty -Name UninstallKey -Value $AppUninstall
     $OutputObj | Add-Member -MemberType NoteProperty -Name AppGUID -Value $AppGUID
    }
   }
  }
 }            
 
 end {}
 }
 
 ################################################
 ################################################
 
 ################################################
 ################################################
 
 Function UninstallProgram {
 
 [cmdletbinding()]            
 
 param (            
 
  [parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true)]
  [string]$ComputerName = $env:computername,
  [parameter(ValueFromPipeline=$true,ValueFromPipelineByPropertyName=$true,Mandatory=$true)]
  [string]$AppGUID
 )            
 
  try {
   $returnval = ([WMICLASS]"\\$computerName\ROOT\CIMV2:win32_process").Create("msiexec `/x$AppGUID `/qn")
  } catch {
   write-error "Failed to trigger the uninstallation. Review the error message"
   $_
   exit
  }
  switch ($($returnval.returnvalue)){
   0 { "Uninstallation command triggered successfully" }
   2 { "You don't have sufficient permissions to trigger the command on $Computer" }
   3 { "You don't have sufficient permissions to trigger the command on $Computer" }
   8 { "An unknown error has occurred" }
   9 { "Path Not Found" }
   9 { "Invalid Parameter"}
  }
 
 }
 ################################################
 ################################################
 
 ################################################
 ################################################
 
 Function FElist {
 $ServerList = Get-Content $Address.text
 $ProgramName = $program.Text
 $OutputListArray= @()
 foreach($Server in $ServerList){
 $ListResults= ListPrograms -ComputerName $Server | Where-Object {$_.AppName -like "*$ProgramName*"} | Select-Object `
 ComputerName, AppName, AppVersion | Format-Table -AutoSize
 $OutputListArray += $ListResults
 }
 $OutputListArray | out-file $env:userprofile\documents\temp_programs.txt
 $ListFile= "$env:userprofile\documents\temp_programs.txt"
 Invoke-Item $ListFile
 }
 
 Function FEuninstall {
 $ServerList = Get-Content $Address.text
 $ProgramName = $program.Text
 $OutputProgArray= @()
 foreach($Server in $ServerList){
 $UninstResult= ListPrograms -ComputerName $Server | Where-Object {$_.AppName -like "*$ProgramName*"} | `
 UninstallProgram
 $OutputProgArray += "$Server $ProgramName " + $UninstResult 
 }
 $OutputProgArray | out-file $env:userprofile\documents\temp_uninst.txt
 $Progfile= "$env:userprofile\documents\temp_uninst.txt"
 Invoke-Item $Progfile
 }
 ################################################
 ################################################
 
 ##############################################################
 ##############################################################
 
     [System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms") 
     [System.Reflection.Assembly]::LoadWithPartialName("System.Drawing") 
      
     $Form = New-Object System.Windows.Forms.Form 
      
     $Form.width = 500
     $Form.height = 300
     $Form.Text = "Remote Uninstall" 
     $Form.maximumsize = New-Object System.Drawing.Size(500, 300) 
     $Form.startposition = "centerscreen" 
     $Form.KeyPreview = $True 
     $Form.Add_KeyDown({if ($_.KeyCode -eq "Escape")  
         {$Form.Close()}}) 
 
     $Infolabel = new-object System.Windows.Forms.Label 
     $Infolabel.Location = new-object System.Drawing.Size(20,10) 
     $Infolabel.size = new-object System.Drawing.Size(450,80) 
     $Infolabel.Font = new-object System.Drawing.Font("Microsoft Sans Serif",10,[System.Drawing.FontStyle]::Bold) 
     $Infolabel.Text = "!! INFORMATION : Make sure you List Programs first. Don't be to vague when specifying the" +
     " Program. For instance if you type Adobe it will unintall all adobe products. Try Adobe Reader and List" +
     " Programs to see what you get. Once you get the correct output when listing then you can uninstall. Uses" + 
     " MSIexec with /qn switch." 
     $Form.Controls.Add($Infolabel) 
      
     $ListButton = new-object System.Windows.Forms.Button 
     $ListButton.Location = new-object System.Drawing.Size(50, 200) 
     $ListButton.Size = new-object System.Drawing.Size(130,30) 
     $ListButton.Text = "List Programs" 
     $ListButton.FlatAppearance.MouseOverBackColor = [System.Drawing.Color]::FromArgb(255, 255, 192); 
     $ListButton.ImageAlign = [System.Drawing.ContentAlignment]::MiddleLeft; 
     $Listbutton.FlatStyle = [System.Windows.Forms.FlatStyle]::Flat 
     $ListButton.Add_Click({FElist}) 
      
     $Form.Controls.Add($ListButton) 
      
     $address = new-object System.Windows.Forms.TextBox 
     $address.Location = new-object System.Drawing.Size(45,160) 
     $address.Size = new-object System.Drawing.Size(220,20) 
      
     $Form.Controls.Add($address) 
      
     $addresslabel = new-object System.Windows.Forms.Label 
     $addresslabel.Location = new-object System.Drawing.Size(45,110) 
     $addresslabel.size = new-object System.Drawing.Size(170,50) 
     $addresslabel.Font = new-object System.Drawing.Font("Microsoft Sans Serif",12,[System.Drawing.FontStyle]::Bold) 
     $addresslabel.Text = "Path to list of Computers:" 
     $Form.Controls.Add($addresslabel) 
      
     $programlabel = new-object System.Windows.Forms.Label 
     $programlabel.Location = new-object System.Drawing.Size(285,110) 
     $programlabel.size = new-object System.Drawing.Size(140,50) 
     $programlabel.Font = new-object System.Drawing.Font("Microsoft Sans Serif",12,[System.Drawing.FontStyle]::Bold) 
     $programlabel.Text = "Program to Uninstall:" 
     $Form.Controls.Add($programlabel) 
      
     $program = new-object System.Windows.Forms.TextBox 
     $program.Location = new-object System.Drawing.Size(285,160) 
     $program.Size = new-object System.Drawing.Size(160,20) 
     
     $Form.Controls.Add($program) 
 
     $UninstallButton = new-object System.Windows.Forms.Button 
     $UninstallButton.Location = new-object System.Drawing.Size(300,200) 
     $UninstallButton.Size = new-object System.Drawing.Size(130,30) 
     $UninstallButton.FlatAppearance.MouseOverBackColor = [System.Drawing.Color]::FromArgb(255, 255, 192); 
     $UninstallButton.ImageAlign = [System.Drawing.ContentAlignment]::MiddleLeft; 
     $UninstallButton.Text = "Uninstall" 
     $Uninstallbutton.FlatStyle = [System.Windows.Forms.FlatStyle]::Flat 
     $UninstallButton.Add_Click({FEuninstall}) 
 
      
     $viclabel = new-object System.Windows.Forms.Label 
     $viclabel.Location = new-object System.Drawing.Size(140,250) 
     $viclabel.size = new-object System.Drawing.Size(200,50) 
     $Form.Controls.Add($viclabel) 
      
     $Form.Controls.Add($UninstallButton) 
      
     $Form.Add_Shown({$Form.Activate()}) 
     $Form.ShowDialog() 
 
 
 ##############################################################
 ##############################################################
`

