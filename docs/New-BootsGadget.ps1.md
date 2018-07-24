---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.5
Encoding: ascii
License: cc0
PoshCode ID: 1546
Published Date: 
Archived Date: 2009-12-26t18
---

# new-bootsgadget - 

## Description

a wrapper for creating gadget windows with powerboots like these

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 New-BootsGadget {
 #.Synopsis
 #.Description
 #.Param Content
 #.Param RefreshRate
 #.Param On_Refresh
 [CmdletBinding(DefaultParameterSetName='DataTemplate')]
 param (
    [Parameter(ParameterSetName='DataTemplate', Mandatory=$False, Position=0)]
    [ScriptBlock]
    ${Content},
    
    [Parameter(Mandatory=$True, Position=1)]
    [TimeSpan][Alias("Rate","Interval")]
    ${RefreshRate},
    
    [Parameter(Mandatory=$True, Position=2)]
    [ScriptBlock]
    ${On_Refresh},
   
    [Parameter(ParameterSetName='FileTemplate', Mandatory=$true, Position=0, HelpMessage='XAML template file')]
    [System.IO.FileInfo]
    ${FileTemplate},
 
    [Parameter(ParameterSetName='SourceTemplate', Mandatory=$true, Position=0, HelpMessage='XAML template XmlDocument')]
    [System.Xml.XmlDocument]
    ${SourceTemplate},
 
    [Parameter(HelpMessage='Do not show in a popup Window (currently only works on PoshConsole)')]
    [Switch]
    ${Inline},
 
    [Parameter(HelpMessage='Write out the window')]
    [Switch]
    ${Passthru},
 
    [Switch]
    ${AllowDrop},
 
    [Switch]
    ${NoTransparency},
 
    [Switch]
    ${ClipToBounds},
 
    [Switch]
    ${Focusable},
 
    [Switch]
    ${ForceCursor},
 
    [Switch]
    ${IsEnabled},
 
    [Switch]
    ${IsHitTestVisible},
 
    [Switch]
    ${IsTabStop},
 
    [Switch]
    ${OverridesDefaultStyle},
 
    [Switch]
    ${ShowActivated},
 
    [Switch]
    ${ShowInTaskbar},
 
    [Switch]
    ${SnapsToDevicePixels},
 
    [Switch]
    ${Topmost},
 
    [Switch]
    ${DialogResult},
 
    [PSObject]
    ${FontSize},
 
    [PSObject]
    ${Height},
 
    [PSObject]
    ${Left},
 
    [PSObject]
    ${MaxHeight},
 
    [PSObject]
    ${MaxWidth},
 
    [PSObject]
    ${MinHeight},
 
    [PSObject]
    ${MinWidth},
 
    [PSObject]
    ${Opacity},
 
    [PSObject]
    ${Top},
 
    [PSObject]
    ${Width},
 
    [PSObject]
    ${TabIndex},
 
    [System.Object]
    ${DataContext},
 
    [System.Object]
    ${ToolTip},
 
    [System.String]
    ${ContentStringFormat},
 
    [System.String]
    ${Name},
 
    [System.String]
    ${Title},
 
    [System.String]
    ${Uid},
 
    [PSObject]
    ${ContextMenu},
 
    [PSObject]
    ${Template},
 
    [PSObject]
    ${ContentTemplateSelector},
 
    [PSObject]
    ${BindingGroup},
 
    [PSObject]
    ${ContentTemplate},
 
    [PSObject]
    ${FlowDirection},
 
    [PSObject]
    ${FontStretch},
 
    [PSObject]
    ${FontStyle},
 
    [PSObject]
    ${FontWeight},
 
    [PSObject]
    ${HorizontalAlignment},
 
    [PSObject]
    ${HorizontalContentAlignment},
 
    [PSObject]
    ${CommandBindings},
 
    [PSObject]
    ${Cursor},
 
    [PSObject[]]
    ${InputBindings},
 
    [PSObject]
    ${InputScope},
 
    [PSObject]
    ${Language},
 
    [PSObject]
    ${BorderBrush},
 
    [PSObject]
    ${Foreground},
 
    [PSObject]
    ${OpacityMask},
 
    [PSObject]
    ${BitmapEffect},
 
    [PSObject]
    ${BitmapEffectInput},
 
    [PSObject]
    ${Effect},
 
    [PSObject]
    ${FontFamily},
 
    [PSObject]
    ${Clip},
 
    [PSObject]
    ${Icon},
 
    [PSObject]
    ${LayoutTransform},
 
    [PSObject]
    ${RenderTransform},
 
    [PSObject]
    ${RenderTransformOrigin},
 
    [PSObject]
    ${Resources},
 
    [PSObject]
    ${RenderSize},
 
    [PSObject]
    ${SizeToContent},
 
    [PSObject]
    ${FocusVisualStyle},
 
    [PSObject]
    ${Style},
 
    [PSObject]
    ${BorderThickness},
 
    [PSObject]
    ${Margin},
 
    [PSObject]
    ${Padding},
 
    [PSObject]
    ${Triggers},
 
    [PSObject]
    ${VerticalAlignment},
 
    [PSObject]
    ${VerticalContentAlignment},
 
    [PSObject]
    ${Visibility},
 
    [PSObject]
    ${Owner},
 
    [PSObject]
    ${WindowStartupLocation},
 
    [PSObject]
    ${On_Closing},
 
    [PSObject]
    ${On_Activated},
 
    [PSObject]
    ${On_Closed},
 
    [PSObject]
    ${On_ContentRendered},
 
    [PSObject]
    ${On_Deactivated},
 
    [PSObject]
    ${On_Initialized},
 
    [PSObject]
    ${On_LayoutUpdated},
 
    [PSObject]
    ${On_LocationChanged},
 
    [PSObject]
    ${On_StateChanged},
 
    [PSObject]
    ${On_SourceUpdated},
 
    [PSObject]
    ${On_TargetUpdated},
 
    [PSObject]
    ${On_ContextMenuClosing},
 
    [PSObject]
    ${On_ContextMenuOpening},
 
    [PSObject]
    ${On_ToolTipClosing},
 
    [PSObject]
    ${On_ToolTipOpening},
 
    [PSObject]
    ${On_DataContextChanged},
 
    [PSObject]
    ${On_FocusableChanged},
 
    [PSObject]
    ${On_IsEnabledChanged},
 
    [PSObject]
    ${On_IsHitTestVisibleChanged},
 
    [PSObject]
    ${On_IsKeyboardFocusedChanged},
 
    [PSObject]
    ${On_IsKeyboardFocusWithinChanged},
 
    [PSObject]
    ${On_IsMouseCapturedChanged},
 
    [PSObject]
    ${On_IsMouseCaptureWithinChanged},
 
    [PSObject]
    ${On_IsMouseDirectlyOverChanged},
 
    [PSObject]
    ${On_IsStylusCapturedChanged},
 
    [PSObject]
    ${On_IsStylusCaptureWithinChanged},
 
    [PSObject]
    ${On_IsStylusDirectlyOverChanged},
 
    [PSObject]
    ${On_IsVisibleChanged},
 
    [PSObject]
    ${On_DragEnter},
 
    [PSObject]
    ${On_DragLeave},
 
    [PSObject]
    ${On_DragOver},
 
    [PSObject]
    ${On_Drop},
 
    [PSObject]
    ${On_PreviewDragEnter},
 
    [PSObject]
    ${On_PreviewDragLeave},
 
    [PSObject]
    ${On_PreviewDragOver},
 
    [PSObject]
    ${On_PreviewDrop},
 
    [PSObject]
    ${On_GiveFeedback},
 
    [PSObject]
    ${On_PreviewGiveFeedback},
 
    [PSObject]
    ${On_GotKeyboardFocus},
 
    [PSObject]
    ${On_LostKeyboardFocus},
 
    [PSObject]
    ${On_PreviewGotKeyboardFocus},
 
    [PSObject]
    ${On_PreviewLostKeyboardFocus},
 
    [PSObject]
    ${On_KeyDown},
 
    [PSObject]
    ${On_KeyUp},
 
    [PSObject]
    ${On_PreviewKeyDown},
 
    [PSObject]
    ${On_PreviewKeyUp},
 
    [PSObject]
    ${On_MouseDoubleClick},
 
    [PSObject]
    ${On_MouseDown},
 
    [PSObject]
    ${On_MouseLeftButtonUp},
 
    [PSObject]
    ${On_MouseRightButtonDown},
 
    [PSObject]
    ${On_MouseRightButtonUp},
 
    [PSObject]
    ${On_MouseUp},
 
    [PSObject]
    ${On_PreviewMouseDoubleClick},
 
    [PSObject]
    ${On_PreviewMouseDown},
 
    [PSObject]
    ${On_PreviewMouseLeftButtonDown},
 
    [PSObject]
    ${On_PreviewMouseLeftButtonUp},
 
    [PSObject]
    ${On_PreviewMouseRightButtonDown},
 
    [PSObject]
    ${On_PreviewMouseRightButtonUp},
 
    [PSObject]
    ${On_PreviewMouseUp},
 
    [PSObject]
    ${On_GotMouseCapture},
 
    [PSObject]
    ${On_LostMouseCapture},
 
    [PSObject]
    ${On_MouseEnter},
 
    [PSObject]
    ${On_MouseLeave},
 
    [PSObject]
    ${On_MouseMove},
 
    [PSObject]
    ${On_PreviewMouseMove},
 
    [PSObject]
    ${On_MouseWheel},
 
    [PSObject]
    ${On_PreviewMouseWheel},
 
    [PSObject]
    ${On_QueryCursor},
 
    [PSObject]
    ${On_PreviewStylusButtonDown},
 
    [PSObject]
    ${On_PreviewStylusButtonUp},
 
    [PSObject]
    ${On_StylusButtonDown},
 
    [PSObject]
    ${On_StylusButtonUp},
 
    [PSObject]
    ${On_PreviewStylusDown},
 
    [PSObject]
    ${On_StylusDown},
 
    [PSObject]
    ${On_GotStylusCapture},
 
    [PSObject]
    ${On_LostStylusCapture},
 
    [PSObject]
    ${On_PreviewStylusInAirMove},
 
    [PSObject]
    ${On_PreviewStylusInRange},
 
    [PSObject]
    ${On_PreviewStylusMove},
 
    [PSObject]
    ${On_PreviewStylusOutOfRange},
 
    [PSObject]
    ${On_PreviewStylusUp},
 
    [PSObject]
    ${On_StylusEnter},
 
    [PSObject]
    ${On_StylusInAirMove},
 
    [PSObject]
    ${On_StylusInRange},
 
    [PSObject]
    ${On_StylusLeave},
 
    [PSObject]
    ${On_StylusMove},
 
    [PSObject]
    ${On_StylusOutOfRange},
 
    [PSObject]
    ${On_StylusUp},
 
    [PSObject]
    ${On_PreviewStylusSystemGesture},
 
    [PSObject]
    ${On_StylusSystemGesture},
 
    [PSObject]
    ${On_PreviewTextInput},
 
    [PSObject]
    ${On_TextInput},
 
    [PSObject]
    ${On_PreviewQueryContinueDrag},
 
    [PSObject]
    ${On_QueryContinueDrag},
 
    [PSObject]
    ${On_RequestBringIntoView},
 
    [PSObject]
    ${On_GotFocus},
 
    [PSObject]
    ${On_Loaded},
 
    [PSObject]
    ${On_LostFocus},
 
    [PSObject]
    ${On_Unloaded},
 
    [PSObject]
    ${On_SizeChanged}
 )
 PROCESS {
    $null = $PSBoundParameters.Remove("RefreshRate")
    $null = $PSBoundParameters.Remove("On_Refresh")
 
    $PSBoundParameters["AllowsTransparency"] = New-Object "Switch" $true
    $PSBoundParameters["Async"] = New-Object "Switch" $true
    $PSBoundParameters["WindowStyle"] = "None"
    $PSBoundParameters["Background"] = "Transparent"
    $PSBoundParameters["On_MouseLeftButtonDown"] = { $this.DragMove() }
    $PSBoundParameters["On_Closing"] = { $this.Tag["Timer"].Stop() }
    $PSBoundParameters["Tag"] = @{"UpdateBlock"=$On_Refresh; "Interval"= $RefreshRate}
    $PSBoundParameters["On_SourceInitialized"] = {
                         $this.Tag["Temp"] = {
                            $this.Interval = [TimeSpan]$this.Tag.Tag.Interval
                            $this.Remove_Tick( $this.Tag.Tag.Temp ) 
                         }
                         $this.Tag["Timer"] = DispatcherTimer -Interval "0:0:02" -On_Tick $this.Tag.UpdateBlock -Tag $this
                         $this.Tag["Timer"].Add_Tick( $this.Tag.Temp )
                         $this.Tag["Timer"].Start()
                      }
    $PSBoundParameters["ResizeMode"] = "NoResize"
    $PSBoundParameters["Passthru"] = $True
    
    $Window = New-BootsWindow @PSBoundParameters
    if($Window) { [Huddled.Dwm]::RemoveFromAeroPeek( $Window.Handle ) }
    if($Passthru) { Write-Output $Window }
 }
 BEGIN {
 try { 
    $null = [Huddled.DWM]
 } catch { 
 Add-Type -Type @"
 using System;
 using System.Runtime.InteropServices;
 
 namespace Huddled {
    public static class Dwm {
       [DllImport("dwmapi.dll", PreserveSig = false)]
       public static extern int DwmSetWindowAttribute(IntPtr hwnd, int attr, ref int attrValue, int attrSize);
 
       [Flags]
       public enum DwmWindowAttribute
       {
          NCRenderingEnabled = 1,
          NCRenderingPolicy,
          TransitionsForceDisabled,
          AllowNCPaint,
          CaptionButtonBounds,
          NonClientRtlLayout,
          ForceIconicRepresentation,
          Flip3DPolicy,
          ExtendedFrameBounds,
          HasIconicBitmap,
          DisallowPeek,
          ExcludedFromPeek,
          Last
       }
 
       [Flags]
       public enum DwmNCRenderingPolicy
       {
          UseWindowStyle,
          Disabled,
          Enabled,
          Last
       }
 
       public static void RemoveFromAeroPeek(IntPtr Hwnd) //Hwnd is the handle to your window
       {
          int renderPolicy = (int)DwmNCRenderingPolicy.Enabled;
          DwmSetWindowAttribute(Hwnd, (int)DwmWindowAttribute.ExcludedFromPeek, ref renderPolicy, sizeof(int));
       }
    }
 }
 "@
 
 [Reflection.Assembly]::Load("UIAutomationClient, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35")
 function global:Select-Window {
 PARAM( [string]$Title="*", 
        [string]$Process="*", 
        [switch]$Recurse,
        [System.Windows.Automation.AutomationElement]$Parent = [System.Windows.Automation.AutomationElement]::RootElement ) 
 PROCESS {
    $search = "Children"
    if($Recurse) { $search = "Descendants" }
    
    $Parent.FindAll( $search, [System.Windows.Automation.Condition]::TrueCondition ) | 
    Add-Member -Type ScriptProperty -Name Title     -Value {
                $this.GetCurrentPropertyValue([System.Windows.Automation.AutomationElement]::NameProperty)} -Passthru |
    Add-Member -Type ScriptProperty -Name Handle    -Value {
                $this.GetCurrentPropertyValue([System.Windows.Automation.AutomationElement]::NativeWindowHandleProperty)} -Passthru |
    Add-Member -Type ScriptProperty -Name ProcessId -Value {
                $this.GetCurrentPropertyValue([System.Windows.Automation.AutomationElement]::ProcessIdProperty)} -Passthru |
 
    Where-Object {
       ((Get-Process -Id $_.ProcessId).ProcessName -like $Process) -and ($_.Title -like $Title)
    }
 }}
 }
 }
`

