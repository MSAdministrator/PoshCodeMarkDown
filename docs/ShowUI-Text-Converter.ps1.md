---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2966
Published Date: 2011-09-24t07
Archived Date: 2016-11-01t13
---

# showui text converter - 

## Description

requires the showui module available to download from here

## Comments



## Usage



## TODO



## module

`convert-texttobinary`

## Code

`#
 #
 $Windowparam = @{
     Width = 500
     Height = 400
     Title = 'Fun Text Converter'
     WindowStartupLocation = 'CenterScreen'
     AsJob = $True
 }
 
 New-Window @Windowparam {
     New-Grid -Rows *,Auto,*,Auto -Children {
         New-TextBox -Row 0 -Name InputBox -TextWrapping Wrap -VerticalScrollBarVisibility Auto
         New-Grid -Row 1 -Columns *,*,Auto,Auto,Auto,*,* -Children {
             New-Button -Column 2 -Name ConvertButton -Width 65 -Height 25 -Content Translate -On_Click {
                 If ($ComboBox.Text -eq 'TextToBinary') {
                     $OutputBox.Text = Convert-TextToBinary $InputBox.Text
                 } ElseIf ($ComboBox.Text -eq 'BinaryToText') {
                     $OutputBox.Text = Convert-BinaryToText $InputBox.Text
                 } ElseIf ($ComboBox.Text -eq 'TextToHex') {
                     $OutputBox.Text = Convert-TextToHex $InputBox.Text
                 } ElseIf ($ComboBox.Text -eq 'HexToText') {
                     $OutputBox.Text = Convert-HexToText $InputBox.Text
                 } ElseIf ($ComboBox.Text -eq 'BinaryToHex') {
                     $OutputBox.Text = Convert-BinaryToHex $InputBox.Text
                 } ElseIf ($ComboBox.Text -eq 'HexToBinary') {
                     $OutputBox.Text = Convert-HexToBinary $InputBox.Text
                 } ElseIf ($ComboBox.Text -eq 'ReverseInput') {
                     $OutputBox.Text = Convert-TextToReverseText $InputBox.Text
                 }
             }
             New-Label -Column 3
             New-ComboBox -Name ComboBox -Column 4 -IsReadOnly:$True -SelectedIndex 0 -Items {
                 New-TextBlock -Text TextToBinary
                 New-TextBlock -Text BinaryToText
                 New-TextBlock -Text TextToHex
                 New-TextBlock -Text HexToText
                 New-TextBlock -Text BinaryToHex
                 New-TextBlock -Text HexToBinary
                 New-TextBlock -Text ReverseInput
             }
         }
         New-TextBox -Row 2 -Name OutputBox -IsReadOnly:$True -TextWrapping Wrap `
         -VerticalScrollBarVisibility Auto
         New-StackPanel -Row 3 -Orientation Horizontal {
             New-Button -Name CopyTextButton -Width 65 -Height 25 -HorizontalAlignment Left -Content CopyText  -On_Click {
                 $OutputBox.text | clip
             }
             New-Label
             New-Button -Name ClearTextButton -Width 65 -Height 25 -HorizontalAlignment Left -Content ClearText -On_Click {
                 $OutputBox.Text=$Null
             }
         }
     }
 } -On_Loaded {
     Function Convert-TextToBinary {
         [cmdletbinding()]
         Param (
             [parameter(ValueFromPipeLine='True')]    
             [string]$Text
         )
         Begin {
             [string[]]$BinaryArray = @()
         }
         Process {
             $textarray = $text.ToCharArray()
             
             ForEach ($a in $textarray) {
                 $BinaryArray += ([convert]::ToString([int][char]$a,2)).PadLeft(8,"0")
             }
         }
         End {
             $BinaryArray -Join " "
         }
     }
 
     Function Convert-BinaryToText {
         [cmdletbinding()]
         Param (
             [parameter(ValueFromPipeLine='True')]
             [string]$Binary
         )
         Begin {
             [string[]]$TextArray = @()        
         }
         Process {
             $BinaryArray = $Binary -split "\s"
             
             ForEach ($a in $BinaryArray) {
                 $TextArray += [char]([convert]::ToInt64($a,2))
             }
         }
         End {
             $TextArray -Join ""        
         }
     }
     Function Convert-TextToHex {
         [cmdletbinding()]
         Param (
             [parameter(ValueFromPipeLine='True')]    
             [string]$Text
         )
         Begin {
             [string[]]$HexArray = @()
         }
         Process {
             $textarr = $text.ToCharArray()
             
             ForEach ($a in $textarr) {
                 $HexArray += "0x$(([convert]::ToString([int][char]$a,16)).PadLeft(8,'0'))"
             }
         }
         End {
             $HexArray -Join " "
         }
     }
 
     Function Convert-HexToText {
         [cmdletbinding()]
         Param (
             [parameter(ValueFromPipeLine='True')]
             [string]$Hex
         )
         Begin {
             [string[]]$textarr = @()        
         }
         Process {
             $HexArray = $Hex -split "\s"
             
             ForEach ($a in $HexArray) {
                 $textarr += [char]([convert]::ToInt64($a.TrimStart('x0'),16))
             }
         }
         End {
             $textarr -join "" 
         }
     }       
 
     Function Convert-HexToBinary {
         [cmdletbinding()]
         Param (
             [parameter(ValueFromPipeLine='True')]
             [string]$Hex
         )
         Begin {
             [string[]]$binarr = @()        
         }
         Process {
             $HexArray = $Hex -split "\s"
             
             ForEach ($a in $HexArray) {
                 $a = ([char]([convert]::ToInt64($a.TrimStart('x0'),16)))
                 $binarr += ([convert]::ToString([int][char]$a,2)).PadLeft(8,"0")
             }
         }
         End {
             $binarr -join " "     
         }
     }
 
     Function Convert-BinaryToHex {
         [cmdletbinding()]
         Param (
             [parameter(ValueFromPipeLine='True')]
             [string]$Binary
         )
         Begin {
             [string[]]$TextArray = @()        
         }
         Process {
             $BinaryArray = $Binary -split "\s"
             
             ForEach ($a in $BinaryArray) {
                 $a = [char]([convert]::ToInt64($a,2))
                 $TextArray += "0x$(([convert]::ToString([int][char]$a,16)).PadLeft(8,'0'))"
             }
         }
         End {
             $TextArray -Join " "     
         }
     }     
 
     Function Convert-TextToReverseText {
         [cmdletbinding()]
         Param (
             [parameter(ValueFromPipeLine='True')]    
             [string]$InputString
         )
         Begin {
         }
         Process {
             $inputarray = $InputString -split ""
         
             [array]::Reverse($inputarray)
         }
         End {
             $inputarray -join ""
         }
     }
 }
`

