---
Author: foureight84
Publisher: 
Copyright: 
Email: 
Version: 18.667
Encoding: ascii
License: cc0
PoshCode ID: 6247
Published Date: 2016-03-08t09
Archived Date: 2016-08-22t02
---

# wpf datagrid xaml - 

## Description

datagrid xaml layout needed for boots datagrid binding example http

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 <Window
 	xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
 	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
 	x:Name="Window"
 	Title="DataGrid Binding"
 	Width="640" Height="480">
 	<Grid >
 		<TextBlock Height="24" Margin="8,8,8,0" TextWrapping="Wrap" Text="DataGrid" VerticalAlignment="Top" FontSize="18.667" FontFamily="Arial" FontWeight="Bold"/>
 		<DataGrid HorizontalAlignment="Left" Margin="8,82,0,8" Width="240" Name="HadesDevices" AutoGenerateColumns="True">
 			<DataGrid.Columns>
 				<DataGridTextColumn Binding="{Binding Path=DeviceGroup}" Header="Manufacturer"></DataGridTextColumn>
 				<DataGridTextColumn Binding="{Binding Path=Device}" Header="Model"></DataGridTextColumn>
 				<DataGridTextColumn Binding="{Binding Path=Platform}" Header="Platform"></DataGridTextColumn>
 			</DataGrid.Columns>
 		</DataGrid>
 	</Grid>
 </Window>
`

