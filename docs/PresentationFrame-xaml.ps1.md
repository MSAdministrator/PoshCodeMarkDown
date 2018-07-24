---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 53.5
Encoding: ascii
License: cc0
PoshCode ID: 2104
Published Date: 
Archived Date: 2010-10-29t00
---

# presentationframe.xaml - 

## Description

a required file for my powershell presentation module

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 <ResourceDictionary
 	xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation" 
 	xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
 	xmlns:Microsoft_Windows_Themes="clr-namespace:Microsoft.Windows.Themes;assembly=PresentationFramework.Aero">
 	<!-- Resource dictionary entries should be defined here. -->
 	<LinearGradientBrush x:Key="NavigationWindowNavigationChromeBackground" EndPoint="0,1" StartPoint="0,0">
 	</LinearGradientBrush>
 	<Style x:Key="NavigationWindowMenu" TargetType="{x:Type Menu}">
 		<Setter Property="ItemsPanel">
 			<Setter.Value>
 				<ItemsPanelTemplate>
 					<DockPanel/>
 				</ItemsPanelTemplate>
 			</Setter.Value>
 		</Setter>
 		<Setter Property="OverridesDefaultStyle" Value="true"/>
 		<Setter Property="KeyboardNavigation.TabNavigation" Value="None"/>
 		<Setter Property="IsMainMenu" Value="false"/>
 		<Setter Property="Template">
 			<Setter.Value>
 				<ControlTemplate TargetType="{x:Type Menu}">
 					<ItemsPresenter/>
 				</ControlTemplate>
 			</Setter.Value>
 		</Setter>
 	</Style>
 	<Style x:Key="MenuScrollButton" BasedOn="{x:Null}" TargetType="{x:Type RepeatButton}">
 		<Setter Property="ClickMode" Value="Hover"/>
 		<Setter Property="MinWidth" Value="0"/>
 		<Setter Property="MinHeight" Value="0"/>
 		<Setter Property="Template">
 			<Setter.Value>
 				<ControlTemplate TargetType="{x:Type RepeatButton}">
 					<DockPanel SnapsToDevicePixels="true" Background="Transparent">
 						<Rectangle x:Name="R1" Fill="Transparent" Width="1" DockPanel.Dock="Right"/>
 						<Rectangle x:Name="B1" Fill="Transparent" Height="1" DockPanel.Dock="Bottom"/>
 						<Rectangle x:Name="L1" Fill="Transparent" Width="1" DockPanel.Dock="Left"/>
 						<Rectangle x:Name="T1" Fill="Transparent" Height="1" DockPanel.Dock="Top"/>
 						<ContentPresenter x:Name="ContentContainer" HorizontalAlignment="Center" Margin="2,2,2,2" VerticalAlignment="Center"/>
 					</DockPanel>
 					<ControlTemplate.Triggers>
 						<Trigger Property="IsPressed" Value="true">
 							<Setter Property="Fill" TargetName="R1" Value="{DynamicResource {x:Static SystemColors.ControlLightLightBrushKey}}"/>
 							<Setter Property="Fill" TargetName="B1" Value="{DynamicResource {x:Static SystemColors.ControlLightLightBrushKey}}"/>
 							<Setter Property="Fill" TargetName="L1" Value="{DynamicResource {x:Static SystemColors.ControlDarkDarkBrushKey}}"/>
 							<Setter Property="Fill" TargetName="T1" Value="{DynamicResource {x:Static SystemColors.ControlDarkDarkBrushKey}}"/>
 							<Setter Property="Margin" TargetName="ContentContainer" Value="3,3,1,1"/>
 						</Trigger>
 					</ControlTemplate.Triggers>
 				</ControlTemplate>
 			</Setter.Value>
 		</Setter>
 	</Style>
 	<Geometry x:Key="UpArrow">M 0,4 L 3.5,0 L 7,4 Z</Geometry>
 	<MenuScrollingVisibilityConverter x:Key="MenuScrollingVisibilityConverter"/>
 	<Geometry x:Key="DownArrow">M 0,0 L 3.5,4 L 7,0 Z</Geometry>
 	<Style x:Key="{ComponentResourceKey ResourceId=MenuScrollViewer, TypeInTargetAssembly={x:Type FrameworkElement}}" BasedOn="{x:Null}" TargetType="{x:Type ScrollViewer}">
 		<Setter Property="HorizontalScrollBarVisibility" Value="Hidden"/>
 		<Setter Property="VerticalScrollBarVisibility" Value="Auto"/>
 		<Setter Property="Template">
 			<Setter.Value>
 				<ControlTemplate TargetType="{x:Type ScrollViewer}">
 					<Grid SnapsToDevicePixels="true">
 						<Grid.ColumnDefinitions>
 							<ColumnDefinition Width="*"/>
 						</Grid.ColumnDefinitions>
 						<Grid.RowDefinitions>
 							<RowDefinition Height="Auto"/>
 							<RowDefinition Height="*"/>
 							<RowDefinition Height="Auto"/>
 						</Grid.RowDefinitions>
 						<Border Grid.Column="0" Grid.Row="1">
 							<ScrollContentPresenter Margin="{TemplateBinding Padding}"/>
 						</Border>
 						<RepeatButton Style="{StaticResource MenuScrollButton}" Focusable="false" Grid.Column="0" Grid.Row="0" Command="{x:Static ScrollBar.LineUpCommand}" CommandTarget="{Binding RelativeSource={RelativeSource TemplatedParent}}">
 							<RepeatButton.Visibility>
 								<MultiBinding FallbackValue="Visibility.Collapsed" Converter="{StaticResource MenuScrollingVisibilityConverter}" ConverterParameter="0">
 									<Binding Path="ComputedVerticalScrollBarVisibility" RelativeSource="{RelativeSource TemplatedParent}"/>
 									<Binding Path="VerticalOffset" RelativeSource="{RelativeSource TemplatedParent}"/>
 									<Binding Path="ExtentHeight" RelativeSource="{RelativeSource TemplatedParent}"/>
 									<Binding Path="ViewportHeight" RelativeSource="{RelativeSource TemplatedParent}"/>
 								</MultiBinding>
 							</RepeatButton.Visibility>
 							<Path Fill="{DynamicResource {x:Static SystemColors.MenuTextBrushKey}}" Data="{StaticResource UpArrow}"/>
 						</RepeatButton>
 						<RepeatButton Style="{StaticResource MenuScrollButton}" Focusable="false" Grid.Column="0" Grid.Row="2" Command="{x:Static ScrollBar.LineDownCommand}" CommandTarget="{Binding RelativeSource={RelativeSource TemplatedParent}}">
 							<RepeatButton.Visibility>
 								<MultiBinding FallbackValue="Visibility.Collapsed" Converter="{StaticResource MenuScrollingVisibilityConverter}" ConverterParameter="100">
 									<Binding Path="ComputedVerticalScrollBarVisibility" RelativeSource="{RelativeSource TemplatedParent}"/>
 									<Binding Path="VerticalOffset" RelativeSource="{RelativeSource TemplatedParent}"/>
 									<Binding Path="ExtentHeight" RelativeSource="{RelativeSource TemplatedParent}"/>
 									<Binding Path="ViewportHeight" RelativeSource="{RelativeSource TemplatedParent}"/>
 								</MultiBinding>
 							</RepeatButton.Visibility>
 							<Path Fill="{DynamicResource {x:Static SystemColors.MenuTextBrushKey}}" Data="{StaticResource DownArrow}"/>
 						</RepeatButton>
 					</Grid>
 				</ControlTemplate>
 			</Setter.Value>
 		</Setter>
 	</Style>
 	<LinearGradientBrush x:Key="NavigationWindowDownArrowFill" EndPoint="0,1" StartPoint="0,0">
 		<LinearGradientBrush.GradientStops>
 			<GradientStopCollection>
 			</GradientStopCollection>
 		</LinearGradientBrush.GradientStops>
 	</LinearGradientBrush>
 	<Style x:Key="NavigationWindowMenuItem" TargetType="{x:Type MenuItem}">
 		<Setter Property="OverridesDefaultStyle" Value="true"/>
 		<Setter Property="Template">
 			<Setter.Value>
 				<ControlTemplate TargetType="{x:Type MenuItem}">
 					<Grid>
 						<Popup x:Name="PART_Popup" AllowsTransparency="true" IsOpen="{Binding (MenuItem.IsSubmenuOpen), RelativeSource={RelativeSource TemplatedParent}}" Placement="Bottom" PopupAnimation="{DynamicResource {x:Static SystemParameters.MenuPopupAnimationKey}}" VerticalOffset="2" Focusable="false">
 							<!-- <Microsoft_Windows_Themes:SystemDropShadowChrome x:Name="Shdw" Color="Transparent"> -->
 								<Border x:Name="SubMenuBorder" Background="{DynamicResource {x:Static SystemColors.MenuBrushKey}}" BorderBrush="{DynamicResource {x:Static SystemColors.ActiveBorderBrushKey}}" BorderThickness="1">
 									<ScrollViewer Style="{DynamicResource {ComponentResourceKey ResourceId=MenuScrollViewer, TypeInTargetAssembly={x:Type FrameworkElement}}}" CanContentScroll="true">
 										<ItemsPresenter Margin="2" KeyboardNavigation.DirectionalNavigation="Cycle" KeyboardNavigation.TabNavigation="Cycle"/>
 									</ScrollViewer>
 								</Border>
 							<!-- </Microsoft_Windows_Themes:SystemDropShadowChrome> -->
 						</Popup>
 						<Grid x:Name="Panel" HorizontalAlignment="Right" Width="26" Background="Transparent">
 								<Border.Background>
 									<LinearGradientBrush EndPoint="0,1" StartPoint="0,0">
 									</LinearGradientBrush>
 								</Border.Background>
 							</Border>
 							<Path x:Name="Arrow" Fill="{StaticResource NavigationWindowDownArrowFill}" Stroke="White" StrokeLineJoin="Round" StrokeThickness="1" HorizontalAlignment="Right" Margin="{TemplateBinding Padding}" VerticalAlignment="Center" SnapsToDevicePixels="false" Data="M 0 0 L 4.5 5 L 9 0 Z"/>
 						</Grid>
 					</Grid>
 					<ControlTemplate.Triggers>
 						<Trigger Property="IsHighlighted" Value="true">
 							<Setter Property="Visibility" TargetName="HighlightBorder" Value="Visible"/>
 						</Trigger>
 						<Trigger Property="IsEnabled" Value="false">
 						</Trigger>
 						<!--
                   <Trigger Property="HasDropShadow" SourceName="PART_Popup" Value="true">
 							<Setter Property="Margin" TargetName="Shdw" Value="0,0,5,5"/>
 							<Setter Property="SnapsToDevicePixels" TargetName="Shdw" Value="true"/>
 						</Trigger>
                   -->
 					</ControlTemplate.Triggers>
 				</ControlTemplate>
 			</Setter.Value>
 		</Setter>
 	</Style>
 	<SolidColorBrush x:Key="CurrentEntryBackground" Opacity="0.25" Color="{StaticResource {x:Static SystemColors.HighlightColorKey}}"/>
 	<Style x:Key="NavigationWindowNavigationButtonJournalEntryStyle" TargetType="{x:Type MenuItem}">
 		<Setter Property="OverridesDefaultStyle" Value="true"/>
 		<Setter Property="Header" Value="{Binding (JournalEntry.Name)}"/>
 		<Setter Property="Command" Value="NavigationCommands.NavigateJournal"/>
 		<Setter Property="CommandTarget" Value="{Binding TemplatedParent, RelativeSource={RelativeSource AncestorType={x:Type Menu}}}"/>
 		<Setter Property="CommandParameter" Value="{Binding RelativeSource={RelativeSource Self}}"/>
 		<Setter Property="JournalEntryUnifiedViewConverter.JournalEntryPosition" Value="{Binding (JournalEntryUnifiedViewConverter.JournalEntryPosition)}"/>
 		<Setter Property="Template">
 			<Setter.Value>
 				<ControlTemplate TargetType="{x:Type MenuItem}">
 					<Grid x:Name="Panel" SnapsToDevicePixels="true" Background="Transparent">
 						<Path x:Name="Glyph" Stroke="{TemplateBinding Foreground}" StrokeEndLineCap="Triangle" StrokeStartLineCap="Triangle" StrokeThickness="2" HorizontalAlignment="Left" Margin="7,5" Width="10" Height="10" SnapsToDevicePixels="false"/>
 						<ContentPresenter Margin="24,5,21,5" ContentSource="Header"/>
 					</Grid>
 					<ControlTemplate.Triggers>
 						<Trigger Property="JournalEntryUnifiedViewConverter.JournalEntryPosition" Value="Current">
 							<Setter Property="Background" TargetName="Panel" Value="{StaticResource CurrentEntryBackground}"/>
 							<Setter Property="Data" TargetName="Glyph" Value="M 0,5 L 2.5,8 L 7,3 "/>
 							<Setter Property="FlowDirection" TargetName="Glyph" Value="LeftToRight"/>
 							<Setter Property="StrokeLineJoin" TargetName="Glyph" Value="Miter"/>
 						</Trigger>
 						<Trigger Property="IsHighlighted" Value="true">
 							<Setter Property="Foreground" Value="{DynamicResource {x:Static SystemColors.HighlightTextBrushKey}}"/>
 							<Setter Property="Background" TargetName="Panel" Value="{DynamicResource {x:Static SystemColors.HighlightBrushKey}}"/>
 						</Trigger>
 						<MultiTrigger>
 							<MultiTrigger.Conditions>
 								<Condition Property="IsHighlighted" Value="true"/>
 								<Condition Property="JournalEntryUnifiedViewConverter.JournalEntryPosition" Value="Forward"/>
 							</MultiTrigger.Conditions>
 							<Setter Property="Stroke" TargetName="Glyph" Value="White"/>
 							<Setter Property="Data" TargetName="Glyph" Value="M 1,5 L 7,5 M 5,1 L 9,5 L 5,9"/>
 						</MultiTrigger>
 						<MultiTrigger>
 							<MultiTrigger.Conditions>
 								<Condition Property="IsHighlighted" Value="true"/>
 								<Condition Property="JournalEntryUnifiedViewConverter.JournalEntryPosition" Value="Back"/>
 							</MultiTrigger.Conditions>
 							<Setter Property="Stroke" TargetName="Glyph" Value="White"/>
 							<Setter Property="Data" TargetName="Glyph" Value="M 9,5 L 3,5 M 5,1 L 1,5 L 5,9"/>
 						</MultiTrigger>
 					</ControlTemplate.Triggers>
 				</ControlTemplate>
 			</Setter.Value>
 		</Setter>
 	</Style>
 	<JournalEntryUnifiedViewConverter x:Key="JournalEntryUnifiedViewConverter"/>
 	<LinearGradientBrush x:Key="NavigationWindowNavigationButtonFillDisabled" EndPoint="0.5,1.0" StartPoint="0.5,0.0">
 		<LinearGradientBrush.GradientStops>
 			<GradientStopCollection>
 			</GradientStopCollection>
 		</LinearGradientBrush.GradientStops>
 	</LinearGradientBrush>
 	<LinearGradientBrush x:Key="NavigationWindowNavigationButtonFillHover" EndPoint="0.5,1.0" StartPoint="0.5,0.0">
 		<LinearGradientBrush.GradientStops>
 			<GradientStopCollection>
 			</GradientStopCollection>
 		</LinearGradientBrush.GradientStops>
 	</LinearGradientBrush>
 	<LinearGradientBrush x:Key="NavigationWindowNavigationButtonFillPressed" EndPoint="0.5,1.0" StartPoint="0.5,0.0">
 		<LinearGradientBrush.GradientStops>
 			<GradientStopCollection>
 			</GradientStopCollection>
 		</LinearGradientBrush.GradientStops>
 	</LinearGradientBrush>
 	<LinearGradientBrush x:Key="NavigationWindowNavigationButtonFillEnabled" EndPoint="0.5,1.0" StartPoint="0.5,0.0">
 		<LinearGradientBrush.GradientStops>
 			<GradientStopCollection>
 			</GradientStopCollection>
 		</LinearGradientBrush.GradientStops>
 	</LinearGradientBrush>
 	<LinearGradientBrush x:Key="NavigationWindowNavigationButtonStrokeEnabled" EndPoint="0,1" StartPoint="0,0">
 		<LinearGradientBrush.GradientStops>
 			<GradientStopCollection>
 			</GradientStopCollection>
 		</LinearGradientBrush.GradientStops>
 	</LinearGradientBrush>
 	<LinearGradientBrush x:Key="NavigationWindowNavigationArrowFill" EndPoint="0,1" StartPoint="0,0">
 		<LinearGradientBrush.GradientStops>
 			<GradientStopCollection>
 			</GradientStopCollection>
 		</LinearGradientBrush.GradientStops>
 	</LinearGradientBrush>
 	<LinearGradientBrush x:Key="NavigationWindowNavigationArrowStrokeEnabled" EndPoint="0,1" StartPoint="0,0">
 	</LinearGradientBrush>
 	<Style x:Key="NavigationWindowBackButtonStyle" TargetType="{x:Type Button}">
 		<Setter Property="OverridesDefaultStyle" Value="true"/>
 		<Setter Property="Command" Value="NavigationCommands.BrowseBack"/>
 		<Setter Property="Focusable" Value="false"/>
 		<Setter Property="Template">
 			<Setter.Value>
 				<ControlTemplate TargetType="{x:Type Button}">
 					<Grid Width="24" Height="24" Background="Transparent">
 						<Ellipse x:Name="Circle" Fill="{StaticResource NavigationWindowNavigationButtonFillEnabled}" Stroke="{StaticResource NavigationWindowNavigationButtonStrokeEnabled}" StrokeThickness="1"/>
 						<Path x:Name="Arrow" Fill="{StaticResource NavigationWindowNavigationArrowFill}" Stroke="{StaticResource NavigationWindowNavigationArrowStrokeEnabled}" StrokeThickness="0.75" HorizontalAlignment="Center" VerticalAlignment="Center" Data="M0.37,7.69 L5.74,14.20 A1.5,1.5,0,1,0,10.26,12.27 L8.42,10.42 14.90,10.39 A1.5,1.5,0,1,0,14.92,5.87 L8.44,5.90 10.31,4.03 A1.5,1.5,0,1,0,5.79,1.77 z"/>
 					</Grid>
 					<ControlTemplate.Triggers>
 						<Trigger Property="IsEnabled" Value="false">
 							<Setter Property="Fill" TargetName="Circle" Value="{StaticResource NavigationWindowNavigationButtonFillDisabled}"/>
 						</Trigger>
 						<Trigger Property="IsMouseOver" Value="true">
 							<Setter Property="Fill" TargetName="Circle" Value="{StaticResource NavigationWindowNavigationButtonFillHover}"/>
 						</Trigger>
 						<Trigger Property="IsPressed" Value="true">
 							<Setter Property="Fill" TargetName="Circle" Value="{StaticResource NavigationWindowNavigationButtonFillPressed}"/>
 						</Trigger>
 					</ControlTemplate.Triggers>
 				</ControlTemplate>
 			</Setter.Value>
 		</Setter>
 	</Style>
 	<Style x:Key="NavigationWindowForwardButtonStyle" TargetType="{x:Type Button}">
 		<Setter Property="OverridesDefaultStyle" Value="true"/>
 		<Setter Property="Command" Value="NavigationCommands.BrowseForward"/>
 		<Setter Property="Focusable" Value="false"/>
 		<Setter Property="Template">
 			<Setter.Value>
 				<ControlTemplate TargetType="{x:Type Button}">
 					<Grid Width="24" Height="24" Background="Transparent">
 						<Ellipse x:Name="Circle" Fill="{StaticResource NavigationWindowNavigationButtonFillEnabled}" Stroke="{StaticResource NavigationWindowNavigationButtonStrokeEnabled}" StrokeThickness="1" Grid.Column="0"/>
 						<Path x:Name="Arrow" Fill="{StaticResource NavigationWindowNavigationArrowFill}" Stroke="{StaticResource NavigationWindowNavigationArrowStrokeEnabled}" StrokeThickness="0.75" HorizontalAlignment="Center" VerticalAlignment="Center" RenderTransformOrigin="0.5,0" Grid.Column="0" Data="M0.37,7.69 L5.74,14.20 A1.5,1.5,0,1,0,10.26,12.27 L8.42,10.42 14.90,10.39 A1.5,1.5,0,1,0,14.92,5.87 L8.44,5.90 10.31,4.03 A1.5,1.5,0,1,0,5.79,1.77 z">
 							<Path.RenderTransform>
 								<ScaleTransform ScaleX="-1"/>
 							</Path.RenderTransform>
 						</Path>
 					</Grid>
 					<ControlTemplate.Triggers>
 						<Trigger Property="IsEnabled" Value="false">
 							<Setter Property="Fill" TargetName="Circle" Value="{StaticResource NavigationWindowNavigationButtonFillDisabled}"/>
 						</Trigger>
 						<Trigger Property="IsMouseOver" Value="true">
 							<Setter Property="Fill" TargetName="Circle" Value="{StaticResource NavigationWindowNavigationButtonFillHover}"/>
 						</Trigger>
 						<Trigger Property="IsPressed" Value="true">
 							<Setter Property="Fill" TargetName="Circle" Value="{StaticResource NavigationWindowNavigationButtonFillPressed}"/>
 						</Trigger>
 					</ControlTemplate.Triggers>
 				</ControlTemplate>
 			</Setter.Value>
 		</Setter>
 	</Style>
 	<ControlTemplate x:Key="FrameNavChromeTemplateKey" TargetType="{x:Type Frame}">
 		<Border Background="{TemplateBinding Background}" BorderBrush="{TemplateBinding BorderBrush}" BorderThickness="{TemplateBinding BorderThickness}" Padding="{TemplateBinding Padding}">
 			<DockPanel>
 				<Grid Height="22" Background="{StaticResource NavigationWindowNavigationChromeBackground}" DockPanel.Dock="Bottom">
 					<Grid.ColumnDefinitions>
 						<ColumnDefinition Width="Auto"/>
 						<ColumnDefinition Width="Auto"/>
 						<ColumnDefinition Width="16"/>
 						<ColumnDefinition Width="*"/>
 					</Grid.ColumnDefinitions>
 					<Menu x:Name="NavMenu" Margin="1,0,0,0" Style="{StaticResource NavigationWindowMenu}" VerticalAlignment="Center" Height="16" Grid.ColumnSpan="3">
 						<MenuItem Style="{StaticResource NavigationWindowMenuItem}" Padding="0,2,4,0" ItemContainerStyle="{StaticResource NavigationWindowNavigationButtonJournalEntryStyle}" IsSubmenuOpen="{Binding (MenuItem.IsSubmenuOpen), Mode=TwoWay, RelativeSource={RelativeSource TemplatedParent}}">
 							<MenuItem.ItemsSource>
 								<MultiBinding Converter="{StaticResource JournalEntryUnifiedViewConverter}">
 									<Binding Path="BackStack" RelativeSource="{RelativeSource TemplatedParent}"/>
 									<Binding Path="ForwardStack" RelativeSource="{RelativeSource TemplatedParent}"/>
 								</MultiBinding>
 							</MenuItem.ItemsSource>
 						</MenuItem>
 					</Menu>
 					<Path StrokeThickness="1" HorizontalAlignment="Left" Margin="2,0,0,0" VerticalAlignment="Center" IsHitTestVisible="false" SnapsToDevicePixels="false" Grid.Column="0" Grid.ColumnSpan="3" Data="M22.5767,21.035 Q27,19.37 31.424,21.035 A12.5,12.5,0,0,0,53.5,13 A12.5,12.5,0,0,0,37.765,0.926 Q27,4.93 16.235,0.926 A12.5,12.5,0,0,0,0.5,13 A12.5,12.5,0,0,0,22.5767,21.035 z">
 						<Path.LayoutTransform>
 							<ScaleTransform ScaleX="0.667" ScaleY="0.667"/>
 						</Path.LayoutTransform>
 						<Path.Stroke>
 							<LinearGradientBrush EndPoint="0,1" StartPoint="0,0">
 								<LinearGradientBrush.GradientStops>
 									<GradientStopCollection>
 									</GradientStopCollection>
 								</LinearGradientBrush.GradientStops>
 							</LinearGradientBrush>
 						</Path.Stroke>
 						<Path.Fill>
 							<LinearGradientBrush EndPoint="0,1" StartPoint="0,0">
 								<LinearGradientBrush.GradientStops>
 									<GradientStopCollection>
 									</GradientStopCollection>
 								</LinearGradientBrush.GradientStops>
 							</LinearGradientBrush>
 						</Path.Fill>
 					</Path>
 					<Button Margin="3,0,1,0" Style="{StaticResource NavigationWindowBackButtonStyle}" Grid.Column="0">
 						<Button.LayoutTransform>
 							<ScaleTransform ScaleX="0.667" ScaleY="0.667"/>
 						</Button.LayoutTransform>
 					</Button>
 					<Button Margin="1,0,0,0" Style="{StaticResource NavigationWindowForwardButtonStyle}" Grid.Column="1">
 						<Button.LayoutTransform>
 							<ScaleTransform ScaleX="0.667" ScaleY="0.667"/>
 						</Button.LayoutTransform>
 					</Button>
 					<!--<TextBlock >-->
 					<TextBlock Grid.Column="3" FontWeight="Bold" Text="{Binding Content.Title, ElementName=PART_FrameCP, Mode=Default}"/>
 					<!--<TextBlock FontWeight="Bold" Text="{Binding BackStack, RelativeSource={RelativeSource FindAncestor, AncestorType={x:Type Frame}}}" />
 					</TextBlock>-->
 				</Grid>
 				<ContentPresenter x:Name="PART_FrameCP"/>
 			</DockPanel>
 		</Border>
 		<ControlTemplate.Triggers>
 			<MultiTrigger>
 				<MultiTrigger.Conditions>
 					<Condition Property="CanGoForward" Value="false"/>
 					<Condition Property="CanGoBack" Value="false"/>
 				</MultiTrigger.Conditions>
 				<Setter Property="IsEnabled" TargetName="NavMenu" Value="false"/>
 			</MultiTrigger>
 		</ControlTemplate.Triggers>
 	</ControlTemplate>
 	<Style TargetType="{x:Type Frame}">
 		<Setter Property="Template">
 			<Setter.Value>
 				<ControlTemplate TargetType="{x:Type Frame}">
 					<Border Background="{TemplateBinding Background}" BorderBrush="{TemplateBinding BorderBrush}" BorderThickness="{TemplateBinding BorderThickness}" Padding="{TemplateBinding Padding}">
 						<ContentPresenter x:Name="PART_FrameCP"/>
 					</Border>
 				</ControlTemplate>
 			</Setter.Value>
 		</Setter>
 		<Style.Triggers>
 			<Trigger Property="NavigationUIVisibility" Value="Visible">
 				<Setter Property="Template" Value="{StaticResource FrameNavChromeTemplateKey}"/>
 			</Trigger>
 			<MultiTrigger>
 				<MultiTrigger.Conditions>
 					<Condition Property="JournalOwnership" Value="OwnsJournal"/>
 					<Condition Property="NavigationUIVisibility" Value="Automatic"/>
 				</MultiTrigger.Conditions>
 				<Setter Property="Template" Value="{StaticResource FrameNavChromeTemplateKey}"/>
 			</MultiTrigger>
 		</Style.Triggers>
 	</Style>
 </ResourceDictionary>
`

