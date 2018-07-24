---
Author: janny
Publisher: 
Copyright: 
Email: 
Version: 1.01
Encoding: ascii
License: cc0
PoshCode ID: 4877
Published Date: 2014-02-04t14
Archived Date: 2014-02-07t23
---

# instances - 

## Description

from greg�s repository on github. plugin for wmiexplorer (copy this file into “plugins” folder in $psscriptroot directory)

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 <?xml version="1.0"?>
 <WmiExplorerPlugin>
   <PluginAuthor>greg zakharov</PluginAuthor>
   <PluginVersion>1.01</PluginVersion>
   <InjectObject>$mnuInst</InjectObject>
   <ObjectText>&amp;Show Instances</ObjectText>
   <Code>
     if (Get-UserStatus) {
       $frmInst = New-Object Windows.Forms.Form
       $rtbInst = New-Object Windows.Forms.RichTextBox
       #
       #
       $rtbInst.Dock = [Windows.Forms.DockStyle]::Fill
       $rtbInst.ReadOnly = $true
       #
       #
       $frmInst.ClientSize = New-Object Drawing.Size(530, 270)
       $frmInst.Controls.Add($rtbInst)
       $frmInst.Icon = $ico
       $frmInst.StartPosition = [Windows.Forms.FormStartPosition]::CenterParent
       $frmInst.Text = "Instances"
       $frmInst.Add_Load({
         try {
           $ins = $wmi.GetInstances()
           
           if ($ins.Count -ne 0) {
             foreach ($i in $ins) {
               $i.PSBase.Properties | % {
                 $rtbInst.SelectionFont = $bol2
                 $rtbInst.AppendText($_.Name + ': ')
                 $rtbInst.SelectionFont = $norm
                 
                 if ($_.Value -eq $null) {
                   $rtbInst.AppendText("`n")
                 }
                 elseif ($_.IsArray) {
                   $ofs = ";"
                   $rtbInst.AppendText("$($_.Value)")
                   $ofs = $null
                   $rtbInst.AppendText("`n")
                 }
                 else {
                   $rtbInst.AppendText("$($_.Value)`n")
                 }
               }
               $rtbInst.AppendText("`n$('=' * 57)`n")
             }
           else {
             $rtbInst.SelectionFont = $bol1
             $rtbInst.AppendText("Out of context.")
           }
         }
         catch [Management.Automation.RuntimeException] {}
       
       [void]$frmInst.ShowDialog()
     }
   </Code>
 </WmiExplorerPlugin>
`

