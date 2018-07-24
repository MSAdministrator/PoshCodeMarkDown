---
Author: ravikanth
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2690
Published Date: 2011-05-23t09
Archived Date: 2011-05-27t22
---

# wacth teched sessions - 

## Description

wpf gui to watch teched 2011, na sessions

## Comments



## Usage



## TODO



## function

`get-techedsession`

## Code

`#
 #
 ipmo WPK
 Function Get-TechEDSession {                
 
     $url = "https://channel9.msdn.com/Events/TechEd/NorthAmerica/2011/RSS/"
     $wc   = New-Object net.webclient
     #$wc.Proxy.Credentials = Get-Credential
     $feed = ([xml]($wc.downloadstring($url))).rss.channel.item            
 
     $feed | ForEach {
                 New-Object PSObject -Property @{
                 Title = $_.title
                 Url = $_.Enclosure.url
                 Category = @($_.Category)
             }
     }
 }                        
 
 $sessions = Get-TechEDSession
 New-Window -Title "TechED - NA 2011 Session viewer" -WindowStartupLocation CenterScreen -Width 1200 -Height 600 -On_Loaded {
     $lstBox = $window | Get-ChildControl lstBox
     $lstBox.ItemsSource = $sessions
 } {
     New-Grid -Rows 32, 1* -Columns 300, 900* {                        
 
         New-ComboBox -Name cmbBox -MaxHeight 400 -MaxWidth 300 -ItemsSource @($sessions | Select -ExpandProperty Category -Unique) -On_SelectionChanged {
             $lstBox = $window | Get-ChildControl lstBox
             $lstBox.ItemsSource = @($sessions | ? { $_.Category -Contains $this.SelectedValue})
         }                        
 
         New-ListBox -Name lstBox -Row 1 -Column 0 -DisplayMemberPath Title -On_SelectionChanged {
                 $wplayer = $window | Get-ChildControl player
                 $wplayer.Source = $this.SelectedItem.Url
             }                        
 
         New-MediaElement -Name player -Column 1 -RowSpan 2
     }
 } -Show
`

