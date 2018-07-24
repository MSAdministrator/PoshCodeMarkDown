---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2252
Published Date: 
Archived Date: 2010-09-26t20
---

# treeview sample - 

## Description

treeview

## Comments



## Usage



## TODO



## function

`view-textfiles`

## Code

`#
 #
 Add-Type -AssemblyName presentationframework 
 
 function view-TextFiles ($dir = '.') {
  
     
   $path = Resolve-Path $dir
   $Form=[Windows.Markup.XamlReader]::Load([IO.File]::OpenText('c:\scripts\window.xaml').basestream) 
   $ListBox = $form.FindName('listBox')
   $RichTextBox = $form.FindName('richTextBox1')
   
 
   ls *.ps1 |% {
     $li = new-Object Windows.Controls.ListBoxItem
     $li.Content = $_.name
     $li.Tag= $_.fullname
     $listbox.Items.Add($LI)
   }
   
   
   $ListBox.add_MouseDoubleClick({
     $RichTextBox.SelectAll()
     $RichTextBox.Selection.Text = gc $this.Selecteditem.Tag
   })
 
   
   $Form.ShowDialog() | out-null
 }
`

