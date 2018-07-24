---
Author: skourlatov
Publisher: 
Copyright: 
Email: 
Version: 1.02
Encoding: ascii
License: cc0
PoshCode ID: 5429
Published Date: 2017-09-15t11
Archived Date: 2017-05-22t01
---

# get-windowsproductgui - 

## Description

function to retrieve windows product info from registry with gui

## Comments



## Usage



## TODO



## function

`get-windowsproductgui`

## Code

`#
 #
 Add-Type -AssemblyName 'PresentationFramework'
 
 {
     function DecodeDigitalPID([byte[]]$digitalProductId)
     {
         $base24 = 'BCDFGHJKMPQRTVWXY2346789'
         $cryptedStringLength = 24
         $decryptionLength    = 14
         $decryptedKey = [string]::Empty
 
         [bool]$containsN = ($digitalProductId[$decryptionLength] -shr 3) -band 1
         [byte]$digitalProductId[$decryptionLength] = $digitalProductId[$decryptionLength] -band 0xF7
 
         for ($i = $cryptedStringLength; $i -ge 0; $i--)
         {
             $digitMapIndex = 0
             for ($j = $decryptionLength; $j -ge 0; $j--)
             {
                 [int]$digitMapIndex = $digitMapIndex -shl 8 -bxor $digitalProductId[$j]
                 [byte]$digitalProductId[$j] = [Math]::Truncate($digitMapIndex / $base24.Length)
                 [int]$digitMapIndex = $digitMapIndex % $base24.Length
             }
             $decryptedKey = $decryptedKey.Insert(0, $base24[$digitMapIndex])
         }
 
         if ($containsN)
         {
             $firstCharIndex = 0
             for ($index = 0; $index -lt $cryptedStringLength; $index++)
             {
                 if ($decryptedKey[0] -ne $base24[$index]) { continue }
                 $firstCharIndex = $index; break
             }
             $decryptedKey = $decryptedKey.Remove(0, 1).Insert($firstCharIndex,'N')
         }
 
         for ($t = 20; $t -ge 5; $t -= 5) { $decryptedKey = $decryptedKey.Insert($t, '-') }
 
         return $decryptedKey
     }
 
     $regKey = Get-ItemProperty -Path 'HKLM:\Software\Microsoft\Windows NT\CurrentVersion'
     return [PSCustomObject]@{
         Version        = [Environment]::OSVersion.VersionString
         Edition        = $regKey.EditionID
         Type        = switch ([Environment]::Is64BitOperatingSystem) { $true {'x86-64'}; $false {'x86-32'}}
         ProductID    = $regKey.ProductId
         ProductKey    = DecodeDigitalPID($regKey.DigitalProductId[0x34..0x42])
         Installed    = [DateTime]::FromBinary(0x089F7FF5F7B58000).AddSeconds($regKey.InstallDate).ToLocalTime().ToString('r').Substring(5,20)
     }
 }
 
 New-Variable -Option 'Constant' -Name 'prod_xaml' -Value ([xml]@"
 <?xml version="1.0" encoding="UTF-8"?>
 <!-- XAML Code - Imported from Visual Studio Express WPF Application -->
 <Window
     xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
     xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
     ResizeMode="NoResize" WindowStyle="None" BorderThickness="0"
     SizeToContent="Width" Height="245" Title="OS Details"
     Topmost="False" FontSize="14">
 
     <Grid Margin="0,0,0,0">
         <Label
             Width="440" Height="30" Margin="0,0,0,0"
             VerticalAlignment="Top" HorizontalAlignment="Center"
             HorizontalContentAlignment="Center" Content="Operating System Details"
             FontWeight="Bold" Name="mainLabel"
         />
         <Button
             Width="30" Height="30" Margin="410,0,0,0" BorderThickness="0"
             VerticalAlignment="Top" HorizontalAlignment="Left"
             FontWeight="Bold" FontSize="20" Content="X"
             Cursor="Hand" ToolTip="Close Window" Name="btnExit"
         />
         <Label Margin="0,35,0,0" Content="Version" Name="lblVersion"/>
         <Label Margin="0,70,0,0" Content="Edition" Name="lblEdition"/>
         <Label Margin="0,105,0,0" Content="Type" Name="lblType"/>
         <Label Margin="0,140,0,0" Content="Product ID" Name="lblProductID"/>
         <Label Margin="0,175,0,0" Content="Product Key" Name="lblProductKey"/>
         <Label Margin="0,210,0,0" Content="Installed" Name="lblInstalled"/>
         <TextBox Margin="115,35,0,0" Name="txtVersion"/>
         <TextBox Margin="115,70,0,0" Name="txtEdition"/>
         <TextBox Margin="115,105,0,0" Name="txtType"/>
         <TextBox Margin="115,140,0,0" Name="txtProductID"/>
         <TextBox Margin="115,175,0,0" Name="txtProductKey"/>
         <TextBox Margin="115,210,0,0" Name="txtInstalled"/>
     </Grid>
 </Window>
 "@)
 
 Function Get-WindowsProductGUI
 {
     $syncHash = [Hashtable]::Synchronized(@{})
     $rsHash   = [Hashtable]::Synchronized(@{})
     $rsHash.Runspace = [RunspaceFactory]::CreateRunspace()
     $rsHash.Runspace.ApartmentState,$rsHash.Runspace.ThreadOptions = 'STA','ReuseThread'
     $rsHash.Runspace.Open()
     $rsHash.Runspace.SessionStateProxy.SetVariable("syncHash",$syncHash)
     $rsHash.Runspace.SessionStateProxy.SetVariable("rsHash",$rsHash)
     $syncHash.xaml = $prod_xaml
 
     $os = Get-WindowsProduct
     $syncHash.Data = New-Object string[](6)
         $syncHash.Data[0] = $os.Version
         $syncHash.Data[1] = $os.Edition
         $syncHash.Data[2] = $os.Type
         $syncHash.Data[3] = $os.ProductId
         $syncHash.Data[4] = $os.ProductKey
         $syncHash.Data[5] = $os.Installed
 
     $psCmd = [PowerShell]::Create().AddScript({
 
         $reader = New-Object -TypeName Xml.XmlNodeReader -ArgumentList $syncHash.xaml
         $Form = [Windows.Markup.XamlReader]::Load($reader)
         $Form.Add_Closed({
             $rshash.PowerShell.Dispose()
 
             [GC]::Collect()
             [GC]::WaitForPendingFinalizers()
         })
         $labels  = New-Object Collections.Generic.List[Windows.Controls.Label]
         $txtblks = New-Object Collections.Generic.List[Windows.Controls.TextBox]
         $syncHash.xaml.SelectNodes("//*[@Name]") | foreach {
 
             $element = $Form.FindName($_.Name)
             switch -Regex ($_.Name)
             {
                      'lbl*' {  $labels.Add($element) }
                      'txt*' { $txtblks.Add($element) }
                   'btnExit' {  $btnExit  = $element  }
                 'mainLabel' {  $lblMain  = $element  }
             }
         }
         $eventHandler_LeftButtonDown = [Windows.Input.MouseButtonEventHandler]{ $Form.DragMove() }
         $lblMain.add_MouseLeftButtonDown($eventHandler_LeftButtonDown)
         $color_shading = {
         }
         $monochrome_shading = {
         }
         $btnExit.add_Click({ $Form.Close() })
         $btnExit.add_MouseEnter({ & $monochrome_shading })
         $btnExit.add_MouseLeave({ & $color_shading })
         for ($i = 0; $i -lt $labels.Count; $i++)
         {
             $tb,$lb = $txtblks[$i],$labels[$i]
 
             $tb.IsReadOnly = $true 
             $tb.Text = $syncHash.Data[$i]
             $tb.Width,$lb.Width = 320,110
             $tb.Height = $lb.Height = 30
             $tb.TextWrapping = [Windows.TextWrapping]::Wrap
             @($tb,$lb) | foreach {
                 $_.VerticalAlignment = [Windows.VerticalAlignment]::Top
                 $_.HorizontalAlignment = [Windows.HorizontalAlignment]::Left
             }
         }
         & $color_shading
         $Form.ShowDialog()
     })
 
     $psCmd.Runspace = $rsHash.Runspace
     [void]$psCmd.BeginInvoke()
 }
`

