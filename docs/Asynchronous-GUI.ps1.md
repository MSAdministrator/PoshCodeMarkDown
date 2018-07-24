---
Author: peter kriegel
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 5520
Published Date: 2016-10-16t11
Archived Date: 2016-12-09t03
---

# asynchronous gui - 

## Description

pattern which shows

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 <#
   How to create a non blocking GUI Thread and how to write Data with PowerShell to GUI controls correctly from a different Runspace.
   
   First of all excuse me for my bad English.
   
   PowerShell is a single threaded application.
   If PowerShell creates a Graphical User Interface (GUI) and should do other things (in the background) it
   can not react on events that are produced by the GUI.
   The user has the impression that the GUI is frozen (blocked).  
   To force PowerShell to do 2 things at the same time you can use PowerShell Jobs or use PowerShell runspaces.
   PowerShell Jobs do not support easy access to Objects inside so we use runspaces to run the GUI.     
   PowerShell runspaces are a kind of a thread inside the PowerShell Process so the word Thread and runspace are used interchangeably here.
   
   Access to Windows Forms controls is not inherently thread safe.
   If you have two or more threads manipulating the state of a control,
   it is possible to force the control into an inconsistent state.
   Other thread-related bugs are possible as well, including race conditions and deadlocks.
   It is important to ensure that access to your controls is done in a thread-safe way.
   
   There was one thing that bothers me all the Time:
   How can I create a non blocking GUI with PowerShell which does not need a synchronized hash table.
   To operate your GUI through a synchronized hash table is like to operate your whole living room through a keyhole
     
   Microsoft suggests the following pattern to make thread-safe calls to Windows Forms controls:
     1. Query the control's InvokeRequired property.
     2. If InvokeRequired returns true, call Invoke with a delegate that makes the actual call to the control.
     3. If InvokeRequired returns false, call the control directly.
   
   See: How to: Make Thread-Safe Calls to Windows Forms Controls
   http://msdn.microsoft.com/en-us/library/ms171728%28v=vs.85%29.aspx
   and dokumentation of the System.Windows.Forms.Control.Invoke or System.Windows.Forms.Control.BeginInvoke methods.
 
   In the fact that we know that we are running the GUI his own thread the use of InvokeRequired is not necessary.
   So we allways use the  System.Windows.Forms.Control.Invoke ore  System.Windows.Forms.Control.BeginInvoke methods here.
 
   Even Windows Presentation Foundation (WPF) uses the Invoke and BeginInvoke Methods of the System.Windows.Threading.Dispatcher class to
   make Thread-Safe calls to WPF controls
 
   Here I present a Pattern how to make thread-safe calls to Windows Forms controls
   I think you can adopt it easily to WPF
   
   Credits are going to:
   Simon Mourier See: http://stackoverflow.com/questions/14401704/update-winforms-not-wpf-ui-from-another-thread
 #>
 
 
 Add-Type -AssemblyName 'System.Windows.Forms'
 Add-Type -AssemblyName 'System.Drawing'
 [Void][Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms.VisualStyles')
 
 
 $Form = New-Object System.Windows.Forms.Form
 $OkButton = New-Object System.Windows.Forms.Button
 $TextBox = New-Object System.Windows.Forms.TextBox
 $Label = New-Object System.Windows.Forms.Label
 $InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
 $Form.Text = 'Demo Form'
 $Form.Name = 'Form'
 $Form.DataBindings.DefaultDataSourceUpdateMode = 0
 $Form.ClientSize =  '156,91'
 
 $TextBox.Size = '100,20'
 $TextBox.DataBindings.DefaultDataSourceUpdateMode = 0
 $TextBox.Name = 'TextBox'
 $TextBox.Location = '27,13'
 
 $Form.Controls.Add($TextBox)
 
 $Label.Location = '27,50'
 $Label.Size = '100,23'
 $Label.Text = 'Round:'
 
 $Form.Controls.Add($Label)
 
 $InitialFormWindowState = $Form.WindowState
 
 $Form.add_Load({
 	$Form.WindowState = $InitialFormWindowState
 })
 
 
 <#
   Create a new runspace
   runspace is an object which is used to configure the PowerShell object
 #>
 $Runspace = [Management.Automation.Runspaces.RunspaceFactory]::CreateRunspace($Host)
 
 
 $Runspace.ApartmentState = 'STA'
 $Runspace.ThreadOptions = 'ReuseThread'
 $Runspace.Open()
 
 $Runspace.SessionStateProxy.SetVariable('Form', $Form)
 
 
 $PowerShellRunspace = [System.Management.Automation.PowerShell]::Create()
 
 $PowerShellRunspace.Runspace = $Runspace
 
 $PowerShellRunspace.AddScript(
      {
      [System.Windows.Forms.Application]::EnableVisualStyles()
      [System.Windows.Forms.Application]::Run($Form)
      })
 
 $AsyncResult = $PowerShellRunspace.BeginInvoke()
 
 
 <#
   Beneath this line you can do your normal PowerShell stuff and you can access the GUI controls
   in a thread-safe manner like it is shown in the examples below.
 #>
 
 
 <#
   block PowerShell Main-Thread to leave it alive until user enter something
   this simulates even that the main-thread is doing hard work
   which normally blocks the GUI Thread
 #>
 $MyMessage = Read-Host 'Enter a message Text'
 
 $TextBox.Invoke(
     <#
     Casting the Scriptblock to an suitable delegate
     The documentation for Control.Invoke (or Control.BeginInvoke) states that:
     The delegate can be an instance of EventHandler, in which case the sender parameter will contain this control,
     and the event parameter will contain EventArgs.Empty.
     The delegate can also be an instance of MethodInvoker, or any other delegate that takes a void parameter list.
     
     System.Action<T> is a delegate and encapsulates a method that
     has a single parameter of type T and does not return a value.
     #>
     [System.Action[string]] { param($Message) $TextBox.Text = $Message },
     $MyMessage
 )
 
 <#
   Control.Invoke is a synchronous call and can be very slow because it waits until the called thread returns
   Better is to use Control.BeginInvoke wich returns immediately a call to
   Control.EndInvoke is optional and makes no sense because all calls should not return anything
 #>
 
 <#
   The documentation for Control.Invoke (or Control.BeginInvoke) even states that:
   A call to an EventHandler or MethodInvoker delegate will be faster than a call to another type of delegate.
   Unfortunately I do not found out how to create an MethodInvoker delegate which takes Parameters so I use the EventHandler delegate.
 #>
 
 $Eventargs = [System.EventArgs]::Empty
 Add-Member -InputObject $eventArgs -MemberType 'NoteProperty' -Name 'MyText' -Value 'Alle Achtung!' -Force
 
 $Handler = [System.EventHandler]{Param($Sender,$Eventargs); $Sender.Text = $Eventargs.MyText }
 
 
 1..50 | ForEach-Object {
   $Eventargs.MyText = "Round: $_"
   $Label.BeginInvoke($Handler,($Label,$Eventargs))
   Start-Sleep -Milliseconds 20
 }
 
 Read-Host 'Enter something to quit'
 
 
 [System.Windows.Forms.Application]::Exit()
 
 $PowerShellRunspace.EndInvoke($AsyncResult)
 
 $Runspace.Close()
 
 $PowerShellRunspace.Dispose()
 
 <#
   Note:
     The call to control.Begininvoke sometime blocks the PowerShell ISE
     I donot se this behavior inside the PowerShell console (PowerShell.exe)  
 #>
`

