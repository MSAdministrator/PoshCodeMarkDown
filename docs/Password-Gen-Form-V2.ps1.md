---
Author: boe prox
Publisher: 
Copyright: 
Email: 
Version: 5.0
Encoding: utf-8
License: cc0
PoshCode ID: 2794
Published Date: 2011-07-16t12
Archived Date: 2016-07-20t23
---

# password gen form v2 - 

## Description

update to original password generator form (http

## Comments



## Usage



## TODO



## script

`create-password`

## Code

`#
 #
 $rs=[RunspaceFactory]::CreateRunspace() 
 $rs.ApartmentState = "STA" 
 $rs.ThreadOptions = "ReuseThread" 
 $rs.Open() 
 $ps = [PowerShell]::Create() 
 $ps.Runspace = $rs 
 $ps.Runspace.SessionStateProxy.SetVariable("pwd",$pwd) 
 [void]$ps.AddScript({  
  
 Add-Type �assemblyName PresentationFramework 
 Add-Type �assemblyName PresentationCore 
 Add-Type �assemblyName WindowsBase 
 Add-Type -AssemblyName System.Windows.Forms 
  
 $printDialog = New-Object System.Windows.Controls.PrintDialog 
  
 Function Create-Password { 
         [int]$passwordlength = 14 
          
         [int]$min_upper = 3 
          
         [int]$min_lower = 3 
          
         [int]$min_number = 3 
          
         [int]$min_symbol = 3 
          
         [int]$min_misc = ($passwordlength - ($min_upper + $min_lower + $min_number + $min_symbol)) 
          
         If ($min_misc -lt 0) { 
             [System.Windows.Forms.MessageBox]::Show("Password specification is not configured correctly, please make the proper edits and try again.","Warning") | Out-Null 
             Break 
             } 
          
         $upper = @("A","B","C","D","E","F","G","H","I","J","K","L","M","N","O","P","Q","R","S","T","U","V","W","X","Y","Z") 
         $lower = @("a","b","c","d","e","f","g","h","i","j","k","l","m","n","o","p","q","r","s","t","u","v","w","x","y","z") 
         $number = @(1,2,3,4,5,6,7,8,9,0) 
         $symbol = @("!","@","#","%","&","(",")","`"",".","<",">","+","=","-","_") 
         $combine = $upper + $lower + $number + $symbol 
          
         $password = @() 
          
         1..$min_upper | ForEach {$password += Get-Random $upper} 
         1..$min_lower | ForEach {$password += Get-Random $lower}  
         1..$min_number | ForEach {$password += Get-Random $number} 
          
         1..$min_symbol | ForEach {$password += Get-Random $symbol}     
          
         1..$min_misc | ForEach {$password += Get-Random $combine}             
          
         $randompassword  = $Null 
          
         Get-Random $password -count $passwordlength | ForEach {[string]$randompassword += $_} 
         Return $randompassword     
     } 
  
 [xml]$xaml = @" 
 <Window 
     xmlns='http://schemas.microsoft.com/winfx/2006/xaml/presentation' 
     xmlns:x='http://schemas.microsoft.com/winfx/2006/xaml' 
     Height = '500' Width = '750' ResizeMode = 'NoResize' WindowStartupLocation = 'CenterScreen'  
     ShowInTaskbar = 'True' Title = 'Password Generator Version 5.0'> 
     <Grid HorizontalAlignment="Stretch" ShowGridLines='false'> 
         <Grid.ColumnDefinitions> 
             <ColumnDefinition Width="25"/> 
             <ColumnDefinition Width="*"/> 
             <ColumnDefinition Width="*"/> 
             <ColumnDefinition Width="25"/> 
         </Grid.ColumnDefinitions> 
         <Grid.RowDefinitions> 
             <RowDefinition Height = '*'/> 
             <RowDefinition Height = 'Auto'/> 
             <RowDefinition Height = '*'/> 
             <RowDefinition Height = '*'/>                 
             <RowDefinition Height = '*'/> 
             <RowDefinition Height = '*'/> 
         </Grid.RowDefinitions> 
         <Label Grid.ColumnSpan = '4' Grid.Column = '0' Grid.Row = '0' HorizontalAlignment = 'Center' Foreground = 'Green' 
         FontWeight="Bold" FontSize="24" VerticalAlignment = 'Center'> 
         FOR OFFICIAL USE ONLY 
         </Label> 
         <TextBlock Grid.ColumnSpan = '2' Grid.Column = '1' Grid.Row = '1' HorizontalAlignment = 'Center' TextWrapping = 'wrap' 
         FontWeight="Bold" FontSize="15"> 
         Ensure your password contains at least 3 special characters, 3 numbers, 3  
         uppercase and 3 lowercase letters for a total of at least 14 characters long. 
         </TextBlock>  
         <Label Grid.Column = '1' Grid.Row = '2' HorizontalAlignment = 'Right' VerticalAlignment = 'Center' FontSize = '16' 
         FontWeight="Bold"> 
         Password: 
         </Label>       
         <TextBox x:Name = 'PassTextBlock' Grid.Column = '2' Grid.Row = '2' HorizontalAlignment = 'left' VerticalAlignment = 'Center' FontSize = '16' 
         FontWeight="Bold" IsReadOnly = 'True' Width = 'Auto'> 
         NOTVALID 
         </TextBox> 
         <CheckBox x:Name = 'PrintCheckBox' Grid.Column = '1' Grid.ColumnSpan = '2' Grid.Row = '3' HorizontalAlignment = 'Center' IsChecked = 'True' 
         VerticalAlignment = 'Center'> 
         Send to Printer 
         </CheckBox> 
         <Button x:Name = 'GenButton' Grid.Column = '1' Grid.ColumnSpan = '2' Grid.Row = '4' HorizontalAlignment = 'Center' Height = '30'> 
         Generate Password 
         </Button>          
         <Label Grid.ColumnSpan = '4' Grid.Column = '0' Grid.Row = '5' HorizontalAlignment = 'Center' Foreground = 'Green' 
         FontWeight="Bold" FontSize="24" VerticalAlignment = 'Center'> 
         FOR OFFICIAL USE ONLY 
         </Label>         
     </Grid>     
 </Window> 
 "@ 
 $reader=(New-Object System.Xml.XmlNodeReader $xaml) 
 $Window=[Windows.Markup.XamlReader]::Load( $reader ) 
  
 $GenButton = $window.FindName('GenButton') 
 $PassTextBlock = $window.FindName('PassTextBlock') 
 $PrintCheckBox = $window.FindName('PrintCheckBox') 
  
 $GenButton.Add_Click({ 
     $PassTextBlock.Text = Create-Password 
     [Windows.Forms.Clipboard]::SetText($PassTextBlock.Text) 
     $window.UpdateLayout() 
     If ($PrintCheckBox.IsChecked) { 
         Try { 
             $printDialog.PrintVisual($window,'Window Print') 
             } 
         Catch { 
             If ($error[0] -match "printqueue") { 
                 [windows.messagebox]::Show('No Default Printer specified!','Warning','OK','Exclamation') 
                 } 
             Else { 
                 [windows.messagebox]::Show('Unknown Error Occurred!','Error','OK','Exclamation') 
                 } 
             } 
          
         } 
     })   
 $Window.Add_Closed({ 
     [Windows.Forms.Clipboard]::Clear() 
     })           
 $window.ShowDialog() | Out-Null 
 }).BeginInvoke()
`

