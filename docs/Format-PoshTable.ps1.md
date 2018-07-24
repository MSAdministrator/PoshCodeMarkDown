---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 1699
Published Date: 2011-03-13t12
Archived Date: 2011-11-16t10
---

# format-poshtable - 

## Description

format-poshtable puts the output in a wpf datagrid (inline in poshconsole, popup otherwise)

## Comments



## Usage



## TODO



## function

`format-poshtable`

## Code

`#
 #
 function Format-PoshTable {
 #.Synopsis
 #.Description 
 	[CmdletBinding()]
 	param(
       [parameter(ValueFromPipeline=$true)]
       [Array]$InputObject
    ,
       [Parameter(Position=1)]
       [String[]]$Property = "*"
    ,
       [Parameter(Position=2)][Alias("Type")]
    ,
       [Parameter()]
       [Switch]$Popup = (![bool]$Host.PrivateData.WpfConsole)
 	)
 	Begin
 	{
       $global:theFormatPoshTableDataGrid = $null
 		if (!(Get-Command datagrid) )
 		{
 			Import-Module PowerBoots
          Add-BootsFunction 'C:\Program Files (x86)\WPF Toolkit\*\WPFToolkit.dll'
 		}
 	}
 	Process
 	{
 		if(!$global:theFormatPoshTableDataGrid) {
          if(!$BaseType) { $BaseType = $InputObject[0].GetType().FullName }
 			$global:ObservableCollection = new-object System.Collections.ObjectModel.ObservableCollection[$BaseType]
          foreach($i in $InputObject) { $ObservableCollection.Add($i) > $null }
          boots {
             Param($ItemCollection, $Property)
 				datagrid -RowBackground "AliceBlue" -AlternatingRowBackground "LightBlue" -On_AutoGeneratingColumn {
                Param($Source,$SourceEventArgs) 
                $header = $SourceEventArgs.Column.Header.ToString()
                $Cancel = $true
                foreach($h in $Source.Tag) {  if($header -like $h) {  $Cancel = $false } }
                $SourceEventArgs.Cancel = $Cancel
             }  -ColumnHeaderStyle {
 					Style -Setters {
 						Setter -Property ([System.Windows.Controls.ListView]::FontWeightProperty) -Value ([System.Windows.FontWeights]::ExtraBold)
 					}
 				} -ItemsSource $ItemCollection -ov global:theFormatPoshTableDataGrid -tag $Property
          } $ObservableCollection $Property -Inline:(!$Popup) -Popup:$Popup
 		} else {
          @($global:theFormatPoshTableDataGrid)[0].Dispatcher.Invoke( "Normal", ([Action]{  foreach($i in @($InputObject)) { $global:ObservableCollection.Add($i) > $null } }) )  
 		}
 	}
 }
`

