---
Author: bielawb
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4022
Published Date: 2013-03-15t21
Archived Date: 2016-03-29t15
---

# localadmingui - 

## Description

gui (wpf) script, that is wrapper on remote endpoint – configuration file for said endpoint can be found in post on my blog explaining it (http

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Add-Type -AssemblyName PresentationCore
 Add-Type -AssemblyName PresentationFramework
 
 [xml]$XAML = @'
 <Window 
     xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
     xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
     Width="316" MinWidth="316" MaxWidth="316"
     Title="Local Admins Tool" SizeToContent="WidthAndHeight" FontSize="14" FontFamily="Consolas" Name="Window">
     <Grid>
         <Grid.ColumnDefinitions>
             <ColumnDefinition Width="Auto"/>
             <ColumnDefinition MinWidth="100"/>
         </Grid.ColumnDefinitions>
         <Grid.RowDefinitions>
             <RowDefinition Height="30"/>
             <RowDefinition Height="30"/>
             <RowDefinition Height="30"/>
             <RowDefinition Height="Auto"/>
         </Grid.RowDefinitions>
         <Label Content="Computer: " Grid.Column="0" />
         <TextBox Grid.Column="1" Name="TextBoxComputer"/>
         <Label Content="Domain: " Grid.Column="0" Grid.Row="1"/>
         <ComboBox Grid.Column="1" Grid.Row="1" Name="ComboDomain">
             <ComboBox.Items>
                 <ComboBoxItem Content="FOO"/>
                 <ComboBoxItem Content="BAR"/>
                 <ComboBoxItem Content="ALP"/>
                 <ComboBoxItem Content="BET"/>
                 <ComboBoxItem Content="GAM"/>
                 <ComboBoxItem Content="DEL"/>
             </ComboBox.Items>
         </ComboBox>
         <StackPanel Grid.ColumnSpan="2" Grid.Row="2" Orientation="Horizontal">
             <Button Name="ButtonList" Content="Check" Width="100"/>
             <Button Name="ButtonAdd" Content="Add" Width="100"/>
             <Button Name="ButtonCancel" Content="Cancel" Width="100"/>
         </StackPanel>
         <ListBox Grid.ColumnSpan="3" Grid.Row="3" Name="AdminsList"/>
         
     </Grid>
 </Window>
 '@
 
 $Reader = New-Object System.Xml.XmlNodeReader $XAML
 $Dialog = [Windows.Markup.XamlReader]::Load($Reader)
 foreach ($Name in ($XAML | Select-Xml '//*/@Name' | foreach { $_.Node.Value})) {
     New-Variable -Name $Name -Value $Dialog.FindName($Name) -Force
 }
 
 $AddScriptBlock = {
     if ($_.Key) {
         if ($_.Key -ne 'Enter') {
             return
         }
     }
 
     $ComputerName = $TextBoxComputer.Text.Trim()
     $AdminsList.Items.Clear()
     $AdminsList.Items.Add(
         (New-Object System.Windows.Controls.ListBoxItem -Property @{
             Content = "Adding admin account to: $ComputerName"
             FontWeight = 'Bold'
         })
     )
 
     if ($ComputerName -notmatch '^[a-z]{3}\d{6}$') {
         $AdminsList.Items.Add(
             (New-Object System.Windows.Controls.ListBoxItem -Property @{
                 Content = "Wrong computer name: $ComputerName"
                 Foreground = 'Red'
             }
         ))
         return
     }
 
     $DomainName = $ComboDomain.SelectedItem.Content
     
     if (!$PredictDomain) {
         $AdminsList.Items.Add(
             (New-Object System.Windows.Controls.ListBoxItem -Property @{
                 Content = "Error: can't find workstation $ComputerName in AD!"
                 Foreground = 'Red'
             }
         ))
         return
     }
 
     if (!$DomainName) {
         $DomainName = $PredictDomain
     }
 
     try {
         Invoke-Command -ComputerName Endpoints -ConfigurationName $PredictDomain -ScriptBlock {
             param ($Computer, $Domain)
             Add-LocalAdmin -ComputerName $Computer -Domain $Domain
         } -ArgumentList $ComputerName, $DomainName -ErrorAction Stop
         $AdminsList.Items.Add(
             (New-Object System.Windows.Controls.ListBoxItem -Property @{
                 Content = 'Done!'
                 Foreground = 'Green'
             })
         )
 
     } catch {
         $AdminsList.Items.Add(
             (New-Object System.Windows.Controls.ListBoxItem -Property @{
                 Content = "Error: $_"
                 Foreground = 'Red'
             }
         ))
     }
 }
 
 $TextChangedScriptBlock = {
     switch -Regex ($this.Text.Trim()) {
         '^[a-z]{3}\d{6}$' {
             $this.Background = 'LawnGreen'
             
             $Searcher = New-Object ADSISearcher -ArgumentList @(
                 [ADSI]'GC://DC=domain,DC=local',
                 "(&(objectClass=computer)(Name=$_))"
             )
             $Global:PredictDomain = $Searcher.FindOne().Path -replace 'GC.*?,DC=([^,]*),.*', '$1'
             $ComboDomain.Items | ForEach-Object {
                 if ($_.Content -eq $PredictDomain) {
                     $ComboDomain.SelectedItem = $_
                 }
             }
             if (!$Global:PredictDomain) {
                 $ComboDomain.SelectedItem = $null
             }
             
         }
         ^$ {
             $this.Background = 'White'
             $Global:PredictDomain = $null
         }
         default {
             $this.Background = 'LightCoral'
             $Global:PredictDomain = $null
         }
     }
 
 }
 
 $ListScriptBlock = {
 
     if ($_.Key) {
         if ($_.Key -ne 'Enter') {
             return
         }
     }
 
     $AdminsList.Items.Clear()
     $ComputerName = $TextBoxComputer.Text
     if ($ComputerName -notmatch '^[a-z]{3}\d{6}$') {
         $AdminsList.Items.Add(
             (New-Object System.Windows.Controls.ListBoxItem -Property @{
                 Content = "Wrong computer name!"
                 Foreground = 'Red'
             }
         ))
         return
     }
 
     if (!$PredictDomain) {
         $AdminsList.Items.Add(
             (New-Object System.Windows.Controls.ListBoxItem -Property @{
                 Content = "Error: can't find workstation $ComputerName in AD!"
                 Foreground = 'Red'
             }
         ))
         return
     }
 
     try {
         $AdminsList.Items.Clear()
         $AdminsList.Items.Add("Administrators on computer $ComputerName :")
         Invoke-Command -ComputerName Endpoints -ConfigurationName $PredictDomain -ScriptBlock {
             param ($Computer)
             Get-LocalAdmin -ComputerName $Computer
         } -ArgumentList $ComputerName -ErrorAction Stop | ForEach-Object {
             $AdminsList.Items.Add($_)
         }
         $AdminsList.Items.Add(
             (New-Object System.Windows.Controls.ListBoxItem -Property @{ 
                 Content = "Done!"
                 ForeGround = 'Green'
             }
         ))
     } catch {
         $AdminsList.Items.Add(
             (New-Object System.Windows.Controls.ListBoxItem -Property @{
                 Content = "Error: $_"
                 Foreground = 'Red'
             }
         ))
     }
 
 }
 
 $TextBoxComputer.Add_KeyDown($ListScriptBlock)
 $ButtonAdd.Add_Click($AddScriptBlock)
 $ButtonList.Add_Click($ListScriptBlock)
 $TextBoxComputer.Add_TextChanged($TextChangedScriptBlock)
 $ButtonCancel.Add_Click({$window.Close()})
 
 $Dialog.Add_Loaded( {
     $this.TopMost = $true
 })
 
 $Dialog.ShowDialog() | Out-Null
 
`

