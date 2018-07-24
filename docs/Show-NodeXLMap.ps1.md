---
Author: doug finke http
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 733
Published Date: 2009-12-15t14
Archived Date: 2017-03-12t01
---

# show-nodexlmap - 

## Description

update version of doug finke’s show-netmap script (http

## Comments



## Usage



## TODO



## function

`add-edge`

## Code

`#
 #
 
 [Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms")   | Out-Null
 [Reflection.Assembly]::LoadWithPartialName("System.Drawing")   | Out-Null
 [Reflection.Assembly]::Loadfrom("$pwd\Microsoft.NodeXL.Control.dll") | Out-Null
 
 function Add-Edge($source, $target, $color)
 {
     [Microsoft.NodeXL.Core.IVertex]$sourceVertex=$null
     $res=$netMapControl.Graph.Vertices.Find($source, [ref] $sourceVertex)
     if ($sourceVertex -eq $null)
     {
 		$sourceVertex = $netMapControl.Graph.Vertices.Add()
 		$sourceVertex.Name = $source
 		$sourceVertex.SetValue("~PVLDPrimaryLabel", $source)
 		if ($color -ne $null)
 		{
 			$sourceVertex.SetValue("~PVLDPrimaryLabelFillColor", [System.Drawing.Color]::$color )
 		}
    	}
 
     [Microsoft.NodeXL.Core.IVertex]$targetVertex=$null
     $res=$netMapControl.Graph.Vertices.Find($target, [ref] $targetVertex)
     if ($targetVertex -eq $null)
     {
 		$targetVertex = $netMapControl.Graph.Vertices.Add()
 		$targetVertex.Name = $target
 		$targetVertex.SetValue("~PVLDPrimaryLabel", $target)
 		if ($color -ne $null)
 		{
 			$targetVertex.SetValue("~PVLDPrimaryLabelFillColor", [System.Drawing.Color]::$color )
 		}
     }
 
     $edge=$netMapControl.Graph.Edges.Add($sourceVertex, $targetVertex, $true)
 }
 
 function Show-NodeXLMap($layoutType="circular") {
   Begin {
     $form = New-Object Windows.Forms.Form
     $netMapControl = New-Object Microsoft.NodeXL.Visualization.NodeXLControl
     $netMapControl.Dock = "Fill"
 	$netMapControl.VertexDrawer = new-object Microsoft.NodeXL.Visualization.PerVertexWithLabelDrawer
     $netMapControl.EdgeDrawer   = new-object Microsoft.NodeXL.Visualization.PerEdgeWithLabelDrawer
     $netMapControl.BeginUpdate()
   }
 
   Process {
     if($_) {
       Add-Edge $_.Source $_.Target $_.Color
     }
   }
 
   End {
     switch -regex ($layoutType) {
        "C" { $Layout = New-Object Microsoft.NodeXL.Visualization.CircleLayout }
        "S" { $Layout = New-Object Microsoft.NodeXL.Visualization.SpiralLayout }
        "H" { $Layout = New-Object Microsoft.NodeXL.Visualization.SinusoidHorizontalLayout }
        "V" { $Layout = New-Object Microsoft.NodeXL.Visualization.SinusoidVerticalLayout }
        "G" { $Layout = New-Object Microsoft.NodeXL.Visualization.GridLayout }
        "F" { $Layout = New-Object Microsoft.NodeXL.Visualization.FruchtermanReingoldLayout }
        "R" { $Layout = New-Object Microsoft.NodeXL.Visualization.RandomLayout }
        "Y" { $Layout = New-Object Microsoft.NodeXL.Visualization.SugiyamaLayout }
     }
 
     $netMapControl.Layout=$layout
     $netMapControl.EndUpdate()
     $form.Controls.Add($NetMapControl)
     $form.WindowState="Maximized"
     #$form.Size = New-Object Drawing.Size @(1000, 600)
     $form.ShowDialog() | out-null
 	
   }
 }
`

