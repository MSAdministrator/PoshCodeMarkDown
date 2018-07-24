---
Author: jrich523
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 5854
Published Date: 2016-05-07t22
Archived Date: 2016-06-30t14
---

# advanced show-object - 

## Description

this is a jacked up version of lee’s show-object. uses wmf and a customized tree view grid (got the c# from msdn some place, lost the link). still pretty early on, will likely update it.

## Comments



## Usage



## TODO



## function

`show-object`

## Code

`#
 #
 
 function Show-Object{
 param(
     [Parameter(ValueFromPipeline = $true)]
     $InputObject
 )
 
 [void][System.Reflection.Assembly]::LoadWithPartialName('presentationframework')
 add-type @"
 using System;
 using System.Windows.Controls;
 using System.Windows;
 using System.Windows.Data;
 using System.Globalization;
 
 
 namespace _treeListView
 {
     public class TreeListView : TreeView
     {
         protected override DependencyObject GetContainerForItemOverride()
         {
             return new TreeListViewItem();
         }
 
         protected override bool IsItemItsOwnContainerOverride(object item)
         {
             return item is TreeListViewItem;
         }
     }
 
     public class TreeListViewItem : TreeViewItem
     {
         private int _level = -1;
         public int Level
         {
             get
             {
                 if (_level != -1) return _level;
                 var parent = ItemsControl.ItemsControlFromItemContainer(this) as TreeListViewItem;
                 _level = (parent != null) ? parent.Level + 1 : 0;
                 return _level;
             }
         }
 
         public TreeListViewItem(object header)
         {
             Header = header;
         }
 
         public TreeListViewItem(){}
 
         protected override DependencyObject GetContainerForItemOverride()
         {
             return new TreeListViewItem();
         }
 
         protected override bool IsItemItsOwnContainerOverride(object item)
         {
             return item is TreeListViewItem;
         }
     }
 
     public class LevelToIndentConverter : IValueConverter
     {
         private const double c_IndentSize = 19.0;
         public object Convert(object o, Type type, object parameter,CultureInfo culture)
         {
             return new Thickness((int)o * c_IndentSize, 0, 0, 0);
         }
 
         public object ConvertBack(object o, Type type, object parameter,CultureInfo culture)
         {
             throw new NotSupportedException();
         }
     }
 
 }
 "@ -ReferencedAssemblies presentationFramework,PresentationCore,WindowsBase,System.Xaml -ErrorAction SilentlyContinue
 [xml]$xaml = @"
 <Window xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
         xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
 	    xmlns:s='clr-namespace:System;assembly=mscorlib' 
 	    xmlns:l='clr-namespace:_treeListView;assembly=$([_treeListView.LevelToIndentConverter].Assembly)' 
 		Title="TreeListView" Width="640" Height="480">
     <Window.Resources>
     <Style x:Key="ExpandCollapseToggleStyle"
            TargetType="{x:Type ToggleButton}">
             <Setter Property="Focusable" Value="False"/>
             <Setter Property="Width" Value="19"/>
             <Setter Property="Height" Value="13"/>
             <Setter Property="Template">
                 <Setter.Value>
                     <ControlTemplate TargetType="{x:Type ToggleButton}">
                         <Border Width="19" Height="13" Background="Transparent">
                                 <Border.Background>
                                     <LinearGradientBrush StartPoint="0,0" EndPoint="1,1">
                                         <LinearGradientBrush.GradientStops>
                                             <GradientStop Color="White" Offset=".2"/>
                                         </LinearGradientBrush.GradientStops>
                                     </LinearGradientBrush>
                                 </Border.Background>
                                 <Path x:Name="ExpandPath" Margin="1,1,1,1" Fill="Black" 
                                       Data="M 0 2 L 0 3 L 2 3 L 2 5 L 3 5 L 3 3 L 5 3 L 5 2 L 3 2 L 3 0 L 2 0 L 2 2 Z"/>
                             </Border>
                         </Border>
                         <ControlTemplate.Triggers>
                             <Trigger Property="IsChecked" Value="True">
                                 <Setter Property="Data" TargetName="ExpandPath" Value="M 0 2 L 0 3 L 5 3 L 5 2 Z"/>
                             </Trigger>
                         </ControlTemplate.Triggers>
                     </ControlTemplate>
                 </Setter.Value>
             </Setter>
         </Style>
 
         <l:LevelToIndentConverter x:Key="LevelToIndentConverter"/>
 
         <DataTemplate x:Key="CellTemplate_Name">
             <DockPanel>
                 <ToggleButton x:Name="Expander" 
                       Style="{StaticResource ExpandCollapseToggleStyle}" 
                       Margin="{Binding Level, Converter={StaticResource LevelToIndentConverter},
                              RelativeSource={RelativeSource AncestorType={x:Type l:TreeListViewItem}}}"
                       IsChecked="{Binding Path=IsExpanded, RelativeSource=
                     {RelativeSource AncestorType={x:Type l:TreeListViewItem}}}"
                       ClickMode="Press"/>
                 <TextBlock Text="{Binding Name}"/>
             </DockPanel>
             <DataTemplate.Triggers>
                 <DataTrigger Binding="{Binding Path=HasItems,RelativeSource={RelativeSource AncestorType={x:Type l:TreeListViewItem}}}" Value="False">
                     <Setter TargetName="Expander" Property="Visibility" Value="Hidden"/>
                 </DataTrigger>
             </DataTemplate.Triggers>
         </DataTemplate>
 
         <GridViewColumnCollection x:Key="gvcc">
             <GridViewColumn Header="Name" CellTemplate="{StaticResource CellTemplate_Name}" />
             <GridViewColumn Header="MemberType" DisplayMemberBinding="{Binding MemberType}" />
             <GridViewColumn Header="Definition" DisplayMemberBinding="{Binding Definition}" />
             <GridViewColumn Header="Value" DisplayMemberBinding="{Binding Value}" />
         </GridViewColumnCollection>
 
         <Style TargetType="{x:Type l:TreeListViewItem}">
             <Setter Property="Template">
                 <Setter.Value>
                     <ControlTemplate TargetType="{x:Type l:TreeListViewItem}">
                         <StackPanel>
                             <Border Name="Bd"
                       Background="{TemplateBinding Background}"
                       BorderBrush="{TemplateBinding BorderBrush}"
                       BorderThickness="{TemplateBinding BorderThickness}"
                       Padding="{TemplateBinding Padding}">
                                 <GridViewRowPresenter x:Name="PART_Header" 
                                       Content="{TemplateBinding Header}" 
                                       Columns="{StaticResource gvcc}" />
                             </Border>
                             <ItemsPresenter x:Name="ItemsHost" />
                         </StackPanel>
                         <ControlTemplate.Triggers>
                             <Trigger Property="IsExpanded" Value="false">
                                 <Setter TargetName="ItemsHost" Property="Visibility" Value="Collapsed"/>
                             </Trigger>
                             <MultiTrigger>
                                 <MultiTrigger.Conditions>
                                     <Condition Property="HasHeader" Value="false"/>
                                     <Condition Property="Width" Value="Auto"/>
                                 </MultiTrigger.Conditions>
                                 <Setter TargetName="PART_Header" Property="MinWidth" Value="75"/>
                             </MultiTrigger>
                             <MultiTrigger>
                                 <MultiTrigger.Conditions>
                                     <Condition Property="HasHeader" Value="false"/>
                                     <Condition Property="Height" Value="Auto"/>
                                 </MultiTrigger.Conditions>
                                 <Setter TargetName="PART_Header" Property="MinHeight" Value="19"/>
                             </MultiTrigger>
                             <Trigger Property="IsSelected" Value="true">
                                 <Setter TargetName="Bd" Property="Background" Value="{DynamicResource {x:Static SystemColors.HighlightBrushKey}}"/>
                                 <Setter Property="Foreground" Value="{DynamicResource {x:Static SystemColors.HighlightTextBrushKey}}"/>
                             </Trigger>
                             <MultiTrigger>
                                 <MultiTrigger.Conditions>
                                     <Condition Property="IsSelected" Value="true"/>
                                     <Condition Property="IsSelectionActive" Value="false"/>
                                 </MultiTrigger.Conditions>
                                 <Setter TargetName="Bd" Property="Background" Value="{DynamicResource  {x:Static SystemColors.ControlBrushKey}}"/>
                                 <Setter Property="Foreground" Value="{DynamicResource {x:Static SystemColors.ControlTextBrushKey}}"/>
                             </MultiTrigger>
                             <Trigger Property="IsEnabled" Value="false">
                                 <Setter Property="Foreground" Value="{DynamicResource {x:Static SystemColors.GrayTextBrushKey}}"/>
                             </Trigger>
                         </ControlTemplate.Triggers>
                     </ControlTemplate>
                 </Setter.Value>
             </Setter>
         </Style>
 
         <Style TargetType="{x:Type l:TreeListView}">
             <Setter Property="Template">
                 <Setter.Value>
                     <ControlTemplate TargetType="{x:Type l:TreeListView}">
                         <Border BorderBrush="{TemplateBinding BorderBrush}" BorderThickness="{TemplateBinding BorderThickness}">
                             <DockPanel>
                                 <GridViewHeaderRowPresenter Columns="{StaticResource gvcc}" DockPanel.Dock="Top"/>
                                 <ScrollViewer CanContentScroll="True">
                                     <Grid>
                                     <ItemsPresenter/>
                                     </Grid>
                                 </ScrollViewer>
                             </DockPanel>
                         </Border>
                     </ControlTemplate>
                 </Setter.Value>
             </Setter>
         </Style>
     </Window.Resources>
     <l:TreeListView x:Name="Tlv"/>
 </Window>
 
 "@
 
 $rootVariableName = dir variable:\* -Exclude InputObject,Args |
     Where-Object {
         $_.Value -and
         ($_.Value.GetType() -eq $InputObject.GetType()) -and
         ($_.Value.GetHashCode() -eq $InputObject.GetHashCode())
 }
 
 $rootVariableName = $rootVariableName| % Name | Select -First 1
 
 if(-not $rootVariableName)
 {
     $rootVariableName = "InputObject"
 }
 
 function PopulateNode($node, $object)
 {
     if(-not $object) { return }
 
     if([System.Management.Automation.LanguagePrimitives]::GetEnumerator($object))
     {
         $isOnlyEnumerable = $object.GetHashCode() -eq $object[0].GetHashCode()
 
         $count = 0
         foreach($childObjectValue in $object)
         {
             $newChildNode = New-Object _treeListView.TreeListViewItem
             $showValue = if(($arr="$childObjectValue" -split "\n" | ?{$_}).count -gt 1)
             {
                 "$($arr[0].trim()) ..."
             }
             else
             {
                 "$childObjectValue"
             }
             $newChildNode.ToolTip = "$childObjectValue"
 
             $newChildNode.Header = [pscustomobject] @{
                 Name=$childObjectValue.GetType().name
                 Value="$showValue"
                 Definition=$childObjectValue.gettype()
                 MemberType="Collection"
             }
             
             if($isOnlyEnumerable)
             {
                 $newChildNode.Tag = "@"
             }
 
             $newChildNode.Tag += "[$count]"
             $null = $node.Items.Add($newChildNode)               
 
             AddPlaceholderIfRequired $newChildNode $childObjectValue
 
             $count++
         }
     }
     else
     {
         $members = Get-Member -InputObject $object
         foreach($member in $members)
         {
             $childNode = New-Object _treeListView.TreeListViewItem
             
             $memberValue = if($member.MemberType -like "*Propert*")
             {
                 
                 $prop = $object.($member.Name)                
                 if($prop)
                 {
                     $prop
                     $childnode.ToolTip = $prop
                     if($prop.gettype().fullname | ?{($_ -split '\.').count -gt 2})
                     {
                         AddPlaceholderIfRequired $childNode $prop
                     }
                 }
                 else { '$null' }
             }
             elseif($member.MemberType -eq "Method")
             {
                $childNode.ToolTip = ($object.($member.name) | Out-String).trim()
             }
 
             $showValue = if(($arr="$memberValue" -split "\n"|?{$_}).count -gt 1)
             {
                 "$($arr[0].trim()) ..."
             }
             else
             {
                 "$memberValue"
             }
             
 
             $childNode.Header = [pscustomobject] @{
                 Name=$member.name
                 Value=$showValue
                 Definition=$member.Definition
                 MemberType=$member.MemberType
             }
 
             $childNode.Tag = $member.Name
             $null = $node.Items.Add($childNode)
         }
     }
 }
 
 function AddPlaceholderIfRequired($node, $object)
 {
     if(-not $object) { return }
 
     if([System.Management.Automation.LanguagePrimitives]::GetEnumerator($object) -or
         @($object.PSObject.Properties))
     {
         $null = $node.Items.Add( (New-Object _treeListView.TreeListViewItem ([pscustomobject]@{Name="..."}) ) )
     }
 }
 
 function OnSelect
 {
     param($Sender, $TreeViewEventArgs)
 
     $nodeSelected = $TreeViewEventArgs.source
 
     $nodePath = GetPathForNode $nodeSelected
 
     $resultObject = Invoke-Expression $nodePath
     #$outputPane.Text = $nodePath
 
     
     if($resultObject)
     {
         $members = Get-Member -InputObject $resultObject | Out-String       
         #$outputPane.Text += "`n" + $members
     }
 }
 
 function OnExpand
 {
     param($Sender, $TreeViewCancelEventArgs)
     $selectedNode = $TreeViewCancelEventArgs.Source
     if($selectedNode.items.Count -eq 1 -and
         ($selectedNode.Items[0].Header.Name-eq "..."))
     {
         $selectedNode.items.Clear()
     }
     else
     {
         return
     }
 
     $nodePath = GetPathForNode $selectedNode 
     $global:nodepath= $nodePath
     Invoke-Expression "`$resultObject = $nodePath"
 
     PopulateNode $selectedNode $resultObject
 }
 
 function OnKeyPress
 {
     param($Sender, $KeyPressEventArgs)
 
     if($KeyPressEventArgs.KeyChar -eq 3)
     {
         $KeyPressEventArgs.Handled = $true
 
         $node = $Sender.SelectedNode
         $nodePath = GetPathForNode $node
         [System.Windows.Forms.Clipboard]::SetText($nodePath)
 
         $form.Close()
     }
 }
 
 function GetPathForNode
 {
     param($Node)
 
     $nodeElements = @()
 
     while($Node.Tag)
     {
         $nodeElements = ,$Node + $nodeElements
         $Node = $Node.Parent
     }
 
     $nodePath = ""
     foreach($Node in $nodeElements)
     {
         $nodeName = $Node.Tag 
 
         if($nodeName.StartsWith('@'))
         {
             $nodeName = $nodeName.Substring(1)
             $nodePath = "@(" + $nodePath + ")"
         }
         elseif($nodeName.StartsWith('['))
         {
         }
         elseif($nodePath)
         {
             $nodePath += "."
         }
 
         $nodePath += $nodeName
     }
 
     $nodePath
 }
 
 
 
 $reader=(New-Object System.Xml.XmlNodeReader $xaml)
 $Form=[Windows.Markup.XamlReader]::Load( $reader )
 
 $tlv = $Form.FindName('Tlv')
 
 
 [System.Windows.RoutedEventHandler]$Script:Select = { OnSelect @args }
 [System.Windows.RoutedEventHandler]$Script:Expand = { OnExpand @args }
 [System.Windows.RoutedEventHandler]$Script:KeyPress = { OnKeyPress @args }
 $tlv.AddHandler([_treeListView.TreeListViewItem]::SelectedEvent, $Script:Select)
 $tlv.AddHandler([_treeListView.TreeListViewItem]::ExpandedEvent, $Script:Expand)
 $tlv.AddHandler([_treeListView.TreeListViewItem]::KeyUpEvent, $Script:KeyPress)
 
 
 $root = New-Object _treeListView.TreeListViewItem ([pscustomobject]@{
     Name=$InputObject.gettype().name
     Value="$InputObject"
     Definition=$InputObject.gettype().fullname})
 
 $root.Tag = '$' + $rootVariableName
 $root.ToolTip = "$InputObject"
 $root.IsExpanded = $true
 
 PopulateNode $root $InputObject
 
 $null = $tlv.Items.Add($root)
 $null = $Form.ShowDialog()
 
 
 }
`

