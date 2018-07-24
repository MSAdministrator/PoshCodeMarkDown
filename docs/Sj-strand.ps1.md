---
Author: david
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 6316
Published Date: 2016-04-22t10
Archived Date: 2016-04-24t06
---

# sj�strand - 

## Description

wpftools is a tool for easy use of wpf

## Comments



## Usage



## TODO



## function

`get-windowsclasses`

## Code

`#
 #
 Add-Type -AssemblyName PresentationFramework
 Add-Type -AssemblyName PresentationCore
 Add-Type �AssemblyName WindowsBase
 
 function Get-WindowsClasses
 {
     $exportedClasses = [System.Reflection.Assembly]::LoadWithPartialName('PresentationFramework').ExportedTypes
 	$exportedClasses += [System.Reflection.Assembly]::LoadWithPartialName('PresentationCore').ExportedTypes
 	$exportedControlClasses = $exportedClasses | Where-Object {$_.isclass -and $_.fullname -like "System.Windows.*"}
 	
 	$controlClasses = @{}
 	foreach ($controlClass in $exportedControlClasses)
 	{
 		$controlClasses[$controlClass.Name] = $controlClass.FullName
 	} 
 	$controlClasses
 }
 
 function Show-WpfWindow ([string]$XamlPath=".\MainWindow.xaml", [string]$HashTableName, [switch]$Dialog = $true)
 {	
 	if (!(Test-Path $XamlPath))
 	{
 		throw "Could not find file $XamlPath"
 	}
     [xml]$xaml = Get-Content $XamlPath
 	$nsmgr = new-object system.xml.xmlnamespacemanager($xaml.nametable)
 	$nsmgr.AddNamespace("x",$xaml.DocumentElement.x)
 	$xaml.window.RemoveAttribute("x:Class")
 	$controlEvents = @{}
 	$controlClasses = Get-WindowsClasses
 	:outer foreach ($element in $xaml.SelectNodes('//*[@x:Name]', $nsmgr))
 	{
 	    $name = $element.name
 	    $typename = $controlClasses[$element.LocalName]
 	    $type = $null
 	    try { $type = [type]$typename } catch { Write-Debug "type $typename does not exist"; continue outer}
 	    Write-Debug "$typeName`: $name"
 	    foreach ($event in $type.GetEvents())
 	    {
 	        $attributeName = $event.Name
 	        $attributeValue = $element.GetAttribute($attributeName)
 	        
 		    if ($attributeValue -and (Test-Path "function:$attributeValue"))
 	        {
                 Write-Debug "Found event: $attributeName - $attributeValue"
 	            $controlEvents[$name] += @{$attributeName=(Get-Item "function:$attributeValue").ScriptBlock}
 	        }
 		    elseif (Test-Path "function:${name}_$attributeName")
 		    {
                 Write-Debug "Found eventhandler $name_$attributeName"
 	            $controlEvents[$name] += @{$attributeName=(Get-Item "function:${name}_$attributeName").ScriptBlock}
 		    }
 		    if ($AttributeValue)
 		    {
 			    $element.RemoveAttribute($attributeName)
 		    }
 	    }
 	    
 	}
 	
 	$reader = New-Object System.Xml.XmlNodeReader($xaml)
 	$Window = [Windows.Markup.XamlReader]::Load( $reader )
     if ($PSBoundParameters.ContainsKey("HashTableName"))
     {
         if (Test-Path "Variable:$HashTableName")
         {
             Remove-Variable $HashTableName
         }
 
         $HashTable = (New-Variable -scope 1 -name $HashTableName -value ([hashtable]::Synchronized(@{})) -PassThru -Option Constant).Value
     }
             
 	foreach ($element in $xaml.SelectNodes('//*[@x:Name]', $nsmgr))
 	{
 	    $name = $element.name
 	    $control = $Window.FindName($name)
 	
 	    if ($control)
 	    {
             if ($PSBoundParameters.ContainsKey("HashTableName"))
             {
                 $HashTable[$name] = $control
             }
             else
             {
                 if (Test-Path "Variable:$name")
                 {
                     Remove-Variable $name
                 }
 	            New-Variable -scope 1 -Name $name -Value $control -Option Constant -ErrorAction SilentlyContinue
             }
 	    }
 	}
 	
 	foreach ($controlName in $controlEvents.Keys)
 	{
 	    $control = $Window.FindName($controlName)
 	    if (!$control)
 	    {
 	        continue
 	    }
 	    foreach ($eventName in $controlEvents[$controlName].Keys)
 	    {
 	        #$scriptBlock = [System.Management.Automation.ScriptBlock]::Create($controlEvents[$controlName][$eventName])
 	        $scriptBlock = $controlEvents[$controlName][$eventName]
             Write-Debug "Regestering event $controlname.${eventName}"
 	        ($control."add_$eventName").Invoke($scriptBlock)
 	    }
 	}
     if ($Dialog)
     {
 	    [void]$Window.ShowDialog()
     }
     else
     {
         [void]$Window.Show()
     }
 }
 
 function Get-WpfSnippet ($XamlPath=".\MainWindow.xaml", [string]$HashTableName,
 	[parameter()]
 	[ValidateSet("None", "Normal", "High", "Full")]
 	[String[]]
 	$CommentDetail="Normal",
 	$CommentBorderLength = 85)
 {
     if ([System.Management.Automation.Runspaces.Runspace]::DefaultRunspace.apartmentstate -ne "STA")
 	{
         throw "Generate-WpfSnippet must be run in a single threaded apartment. Start PowerShell with the -STA flag and rerun the script."
         exit
     }
 
 	[xml]$xaml = Get-Content $XamlPath
 	$nsmgr = new-object system.xml.xmlnamespacemanager($xaml.nametable)
 	$nsmgr.AddNamespace('x',$xaml.DocumentElement.x)
 	#$xaml.window.RemoveAttribute("x:Class")
         if ($CommentDetail -ne 'None')
         {
 	    '#' * $CommentBorderLength
 	    '#'
         }
 
 	$controlEvents = @{}
 	$controlTypes = @{}
 	$controlClasses = Get-WindowsClasses
 	:outer foreach ($element in $xaml.SelectNodes('//*[@x:Name]', $nsmgr))
 	{
 	    $name = $element.name
 	    $typename = $controlClasses[$element.LocalName]
 	    Write-Debug "$typeName`: $name"
 	    $type = $null
         if ($controlTypes.ContainsKey($typename))
         {
             $type = $controlTypes[$typename]
         }
         else
         {
 	        try { $type = [type]$typename } catch { Write-Debug "Unknown error getting type $typename"; continue outer}
             $controlTypes[$typename] = $type
         }
         
         if ($CommentDetail -ne 'None')
         {
             if ($PSBoundParameters.ContainsKey("HashTableName"))
             {
             }
             else
             {
             }
 	    }
 	    foreach ($event in $type.GetEvents())
 	    {
 	        $attributeName = $event.Name
 	        $attributeValue = $element.GetAttribute($attributeName)
 		    if ($attributeValue)
 	        {
 	            Write-Debug "Found event handler: $attributeName - $attributeValue"
 	            $controlEvents[$attributeValue] += @{$name=$attributeName}
 	            #$element.RemoveAttribute($attributeName)
 	        }
 	    }
 	    
 	}
     if ("High","Full" -contains $CommentDetail)
     {
 	    '#' * $CommentBorderLength
 	    '#'
 	    foreach ($typename in $controlTypes.Keys)
 	    {
 		    $count = 0
             foreach ($eventName in ($controlTypes[$typename].GetEvents() | Select-Object -ExpandProperty Name))
             {
                 if (!($count++ % 3))
                 {
                 }
                 $str += "$eventName ".PadRight(30)
             }
 		    $str.Split("`n")
 		    if ($CommentDetail -ne "Full")
 		    {
 		        continue
             }
             foreach ($Property in $controlTypes[$typename].GetProperties())
             {
             }
 
         }
     } 
     if ($CommentDetail -ne 'None')
     {
 	    '#'
 	    '#' * $CommentBorderLength
             ''
     }
     '. .\wpftools.ps1'
 	''
 	''
 	foreach ($eventName in $controlEvents.Keys)
 	{
 	    foreach ($controlName in $controlEvents[$eventName].Keys)
 	    {
         }
 	    "function $eventName"
         '{'
         '}'
         ''	    
 	}
 	
     if ($PSBoundParameters.ContainsKey("HashTableName"))
     {
         "Show-WpfWindow -XamlPath '$XamlPath' -HashTableName '$HashTableName'"
     }
     else
     {
 	    "Show-WpfWindow -XamlPath '$XamlPath'"
     }
 }
`

