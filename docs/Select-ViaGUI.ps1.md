---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2820
Published Date: 2011-07-24t21
Archived Date: 2015-12-28t18
---

# select-viagui - 

## Description

an interactive graphical filter for pipeline objects for showui

## Comments



## Usage



## TODO



## function

`select-viaui`

## Code

`#
 #
 function Select-ViaUI {
 #.Synopsis
 #.Description
 #.Example
     [OutputType([Windows.Controls.Grid])]
     [CmdletBinding(DefaultParameterSetName="DefaultView")]
     param(
     #
     #
     [Parameter(Position=0, ParameterSetName="Property")]
     [Object[]]$Property,
 
     [Parameter(ParameterSetName="View")]
     [String]$View,
     
     [Parameter(ValueFromPipeline=$true)]
     [PSObject[]]$InputObject,
     
     [string]$Name,
     [Int]$Row,
     [Int]$Column,
     [Int]$RowSpan,
     [Int]$ColumnSpan,
     [Int]$Width, 
     [Int]$Height,
     [Double]$Top,
     [Double]$Left,
     [Windows.Controls.Dock]$Dock,
     [Switch]$Show,
     [Switch]$AsJob
 )
 
 begin {
    $Items = New-Object System.Collections.ArrayList
 }
 process {
    $Items.AddRange($InputObject)
 }
 end {
     $uiParameters = @{} + $psBoundParameters
     $null = $uiParameters.Remove("Property")
     $null = $uiParameters.Remove("View")
     $null = $uiParameters.Remove("InputObject")
     
     $formatParameters = @{}
     if($psBoundParameters.ContainsKey("Property")){
         $formatParameters.Property = $PsBoundParameters.Property + @{Name="OriginalItem";Expression={$_}}
     }
     if($psBoundParameters.ContainsKey("View")){
         $formatParameters.View = $psBoundParameters.View
     }
     
     $global:SelectViaUIStringItems = New-Object System.Collections.ArrayList
     if($psBoundParameters.ContainsKey("Property")) {
         $SelectViaUIStringItems = $items | Select-Object @formatParameters
         $Strings = $Items | Format-Table @formatParameters -AutoSize | Tee-Object -variable formattedData | Out-String -Width 10000 -Stream
         $tableColumnInfo = $formattedData[0].shapeInfo.tableColumnInfoList | 
                     Select-Object width, alignment, @{n="label";e={if($_.label){$_.label}else{$_.propertyName}}} | 
                     Where-Object { $_.label -ne "OriginalItem" } 
     } else {
         Write-Warning $psBoundParameters.Keys
         $Strings = $Items | Format-Table | Tee-Object -variable formattedData | Out-String -Width 10000 -Stream
         $tableColumnInfo = $formattedData[0].shapeInfo.tableColumnInfoList | Select-Object width, alignment, @{n="label";e={if($_.label){$_.label}else{$_.propertyName}}}
         for($c=0;$c -lt $Strings.Length;$c++) {
             if( $Strings[$c] -match '^ *-+[ -]*$' ) {
                 $separators = [regex]::Matches($Strings[$c],"(?<=^|\s+)-+") + (New-Object PSObject -Property @{Index =$Delimiters.Length})
                 break
             }
         }
         $Strings = $Strings[($c+1)..($Strings.Count -2)] | where-object { $_ }
 
         for($i=0; $i -lt $Items.Count;$i++) {
             $start = 0
             $line = $Strings[$i]
             $outputRow = @{} 
             for($c=0;$c -lt $tableColumnInfo.Count;$c++) {
                 $length = $tableColumnInfo[$c].width
                 if(!$length) { 
                     if($tableColumnInfo.Count -gt $c) {
                         if($tableColumnInfo[$c].alignment -eq 3) {
                             $length = $start - ($separators[$c].index + $separators[$c].length)
                         } elseif($tableColumnInfo[$c+1].alignment -eq 1) {
                             $length = $Start - $separators[$c+1].index
                         } else {
                             $length = $start - ($separators[$c].index + $separators[$c].length)
                         }
                     }
                     $length = $tableColumnInfo[$c].width = $line.Length - $start
                 }
                 $outputRow.($tableColumnInfo[$c].label) = $line.substring($start, $length).Trim()
                 $start += $length + 1
             }
             $outputRow = New-Object PSObject -Property $outputRow
             $outputRow | Add-Member -Type NoteProperty -Name OriginalItem -Value $Items[$i]
             $null = $SelectViaUIStringItems.Add( $outputRow )
         }
     }
     $SelectViaUIStringItems | Add-Member -Type ScriptMethod -Name ToString -Value { ($this.OriginalItem | Format-Table @formatParameters -HideTableHeaders | Out-String -Width 10000).Trim() } -Force
 
 Grid -Margin 5  -ControlName SelectFTList -Rows Auto, *, Auto, Auto -Children {
     TextBlock -Margin 5 -Row 0 "Type or click to search. Press Enter or click OK to pass the items down the pipeline." 
 
     ScrollViewer -Margin 5 -Row 1 {
         ListView -SelectionMode Extended -ItemsSource $SelectViaUIStringItems -Name SelectedItems  `
                 -FontFamily "Consolas, Courier New" -View {
                     GridView {
                        foreach($h in $tableColumnInfo) {
                            GridViewColumn -Header $h.label -DisplayMember { Binding $h.Label }
                        }
                     }
                 } -On_SelectionChanged {
                     if($selectedItems.SelectedItems.Count -gt 0)
                     {
                         $SelectFTList | Set-UIValue -value ( $selectedItems.SelectedItems | ForEach-Object { $_.OriginalItem } )
                     } else {
                         $SelectFTList | Set-UIValue -value ( $selectedItems.Items | ForEach-Object { $_.OriginalItem } )
                     }
                 } -On_Loaded {
                     $SelectFTList | Set-UIValue -value ( $selectedItems.Items | ForEach-Object { $_.OriginalItem } )
                 }
     }
 
     TextBox -Margin 5 -Name SearchText -Row 2 -On_KeyUp {
         $filterText = $this.Text
         [System.Windows.Data.CollectionViewSource]::GetDefaultView( $SelectedItems.ItemsSource ).Filter = [Predicate[Object]]{ 
             param([string]$item)
             trap { return $true }
             $item -match $filterText
         }
         
         if($selectedItems.SelectedItems.Count -gt 0)
         {
             $SelectFTList | Set-UIValue -value ( $selectedItems.SelectedItems | ForEach-Object { $_.OriginalItem } )
         } else {
             $SelectFTList | Set-UIValue -value ( $selectedItems.Items | ForEach-Object { $_.OriginalItem } )
         }
     }
 
     Grid -Margin 5 -HorizontalAlignment Right -Columns 65, 10, 65 {
         Button "OK" -IsDefault -Width 65 -On_Click { Close-Control $window } -Column 0
         Button "Cancel" -IsCancel -Width 65 -On_Click { $SelectFTList | Set-UIValue -value $null } -Column 2
     } -Row 3
 } -On_Loaded { 
     $SearchText.Focus()
 } @uiParameters
 
 }}
`

