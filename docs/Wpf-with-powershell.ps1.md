---
Author: david sjstrand
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: utf-8
License: cc0
PoshCode ID: 5454
Published Date: 2014-09-19t07
Archived Date: 2014-09-25t15
---

# wpf with powershell - 

## Description

i know there already are a bunch of approaches to this, but this is my somewhat hacky contribution to wpf with powershell. i wanted to be able to draw my ui in visual studio or blend and then use the xaml with my powershell script in a simple way. it certainly has room for improvements, but i think it has potential. i’m quite sure that there are other (nicer) ways to do some of the more hacky stuff.

## Comments



## Usage



## TODO



## function

`get-runspace`

## Code

`#
 #
 Add-Type -Assembly PresentationFramework
 Add-Type -Assembly PresentationCore
 
 function Get-Runspace ($ScriptPath)
 {
 	if ($runspaceCreated -or [System.Management.Automation.Runspaces.Runspace]::DefaultRunspace.apartmentstate -eq "STA")
 	{
 		Write-Debug "No new runspace was created"
 		return
 	}
 
 	if ($PSBoundParameters.ContainsKey('ScriptPath'))
 	{
 		$ScriptPath = Resolve-Path $ScriptPath
 	}
 	elseif ($host.version.major -ge 3) 
 	{
 		$ScriptPath = $MyInvocation.PSCommandPath
 	} 
 	else
 	{
 		$ScriptPath = Resolve-Path (Get-PSCallStack)[-2].InvocationInfo.InvocationName
 	} 
 
 	Write-Debug "Script path: $ScriptPath"
 	Write-Debug "Creating a new STA runspace ..."
 	$rs = [Management.Automation.Runspaces.RunspaceFactory]::CreateRunspace($Host)
 	$rs.ApartmentState = "STA"
 	$rs.ThreadOptions = �ReuseThread�
 	$rs.Open()
 	$rs.SessionStateProxy.SetVariable("runspaceCreated",$true)
 	$rs.SessionStateProxy.SetVariable("debugPreference",$debugPreference)
 	$psCmd = [System.Management.Automation.PowerShell]::Create()
 	$psCmd.Runspace = $rs
 	Write-Debug "Rerunning $ScriptPath"
 	[void]$psCmd.AddCommand("Set-Location")
 	[void]$pscmd.AddParameter("Path",(Get-Location).Path)
 	[void]$psCmd.AddScript($ScriptPath)
 	[void]$psCmd.Invoke()
 	$rs.Close()
 	exit
 }
 
 function Get-WindowsClasses
 {
 	$exportedClasses = [System.Reflection.Assembly]::GetAssembly([System.Windows.Window]).exportedTypes
 	$exportedControlClasses = $exportedClasses | Where-Object {$_.isclass -and $_.fullname -like "System.Windows.*"}
 	
 	$controlClasses = @{}
 	foreach ($controlClass in $exportedControlClasses)
 	{
 		$controlClasses[$controlClass.Name] = $controlClass.FullName
 	} 
 	$controlClasses
 }
 
 function Show-WpfWindow ([string]$XamlPath=".\MainWindow.xaml", [string]$HashTableName)
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
 	    Write-Debug "$typeName`: $name ($(@($type.GetEvents()).count))"
 	    foreach ($event in $type.GetEvents())
 	    {
 	        $attributeName = $event.Name
 	        $attributeValue = $element.GetAttribute($attributeName)
 	        
 		    if ($attributeValue -and (Test-Path "function:$attributeValue"))
 	        {
                 Write-Debug "Found event handler: $attributeName - $attributeValue"
 	            $controlEvents[$name] += @{$attributeName=$attributeValue}
 	        }
 		    elseif (Test-Path "function:${name}_$attributeName")
 		    {
                 Write-Debug "Found eventhandler $name_$attributeName"
 	            $controlEvents[$name] += @{$attributeName="${name}_$attributeName"}
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
 
         $HashTable = (new-variable -name $HashTableName -value @{} -PassThru -Option Constant).Value
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
 	            New-Variable -Name $name -Value $control -Option Constant -ErrorAction SilentlyContinue
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
 	        $scriptBlock = [System.Management.Automation.ScriptBlock]::Create($controlEvents[$controlName][$eventName])
 	        Write-Debug "$controlname.add_$eventName"
 	        ($control."add_$eventName").Invoke($scriptBlock)
 	    }
 	}
 	[void]$Window.ShowDialog()
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
         throw "Get-WpfSnippet must be run in a single threaded apartment. Start PowerShell with the -STA flag and rerun the script."
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
 	'Get-Runspace $MyInvocation.MyCommand.Definition'
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

