---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2371
Published Date: 
Archived Date: 2010-11-23t15
---

# boots hierarchical bind - 

## Description

example of how to use treeview hierarchical databind using boots. it took me a while to figure this out. thanks jasonmarcher for your help!

## Comments



## Usage



## TODO



## module

``

## Code

`#
 #
 
 
 City,Team
 "Los Angeles","Lakers"
 "Los Angeles","Clippers"
 "New York","Knicks"
 "New York","Liberty"
 "Sacramento","Kings"
  
  
 $teams = Import-Csv "C:\testdata.csv"
 
 
 [array]$cities = $teams | %{$_.City} | Sort -Unique
 
 
 [Object[]]$test = foreach ($city in $cities){
         New-Object psobject -Property @{
                 City = $city
                 Teams = @(foreach($team in $($teams | ?{$_.City -eq $city} | %{$_.Team} | Sort -Unique)){
                         New-Object psobject -Property @{
                         Team = $Team
                         IsSelected = "True"
                         }
                 })
         }
 }
  
 Import-Module PowerBoots
 
 $xaml = @"
 <Window
         xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
         xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
         x:Name="Window">
         <Grid>
                 <TreeView Name="treeview1">
                         <TreeView.ItemTemplate>
                                 <HierarchicalDataTemplate ItemsSource="{Binding Teams}">
                                         <TextBlock Foreground="Green" Text="{Binding City}" />
                                         <HierarchicalDataTemplate.ItemTemplate>
                                                 <DataTemplate>
                                                         <StackPanel Orientation="Horizontal">
                                                                 <TextBlock Text="{Binding Team}" />
                                                                 <CheckBox IsChecked="{Binding IsSelected}" IsEnabled="False"/>
                                                         </StackPanel>
                                                 </DataTemplate>
                                         </HierarchicalDataTemplate.ItemTemplate>
                                   
                                 </HierarchicalDataTemplate>
                         </TreeView.ItemTemplate>
                 </TreeView>
         </Grid>
 </Window>
 "@
  
 $MainWindow= New-BootsWindow -Title "Treeview Binding" -Width 100 -Height 150 -SourceTemplate $xaml -Async -Passthru -On_Loaded {
         Export-NamedElement
         $treeView1.ItemsSource = $test
 }
`

