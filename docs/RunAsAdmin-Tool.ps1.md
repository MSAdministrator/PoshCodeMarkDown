---
Author: cody dean
Publisher: 
Copyright: 
Email: 
Version: 1.0.10.0
Encoding: utf-8
License: cc0
PoshCode ID: 4025
Published Date: 2013-03-17t18
Archived Date: 2013-03-21t06
---

# runasadmin tool - 

## Description

runasadmin is an application to help adhere to the best practice of least privileges.  it creates an .exe that the user can double click and it will execute a program with elevated privileges.  users must be in a local group in order to execute the process.  for complete details visit

## Comments



## Usage



## TODO



## script

`get-script`

## Code

`#
 #
 
 DOWNLOAD THE DEPLOY PACKAGE FROM http://techjeeper.com/?page_id=112 for all the pieces to make this work. 
 
 Author: Cody Dean (aka TechJeeper)
 Date: 3-16-2013
 
 RunAsAdmin uses (and includes) the following tools:
 PowerShell Community Extensions
 SysInternals PsExec
 PSTerminalServices
 Make-ps1exewrapper.ps1
 PowerShellPack
 
 To implement a new RunAsAdmin program, extract the RunAsAdmin.zip to the C:\Tools\ directory.
 
 From an Administrator PowerShell prompt do the following:
 
 PS> cd C:\Tools\
 PS C:\Tools\> .\Deploy-RunAsAdmin.ps1
 
 The Deploy RunAsAdmin window will appear.
 
 GUI Description: 
 Application Path: The path to the application you want to run with elevated privileges.
 Task Name: The name given to the application (will be the name of the exe the user will run)
 Project Short-Name:  A �name� for the project, allows for multiple programs to utilize a single group for user access.
 Service Account Name:  The name of the Service Account that will be an Administrator and will run the program.  If the SA doesn�t exist, it will be created.
 Service Account Password:  The password for the SA.  If the SA exists, password must match existing password.
 Cleanup Checkbox: If checked, the source files will be deleted, leaving only the files for the process in C:\Tools.
 #>
 
 $winVersion = (Get-WmiObject Win32_OperatingSystem).Caption
 $psVersion = $Host.Version
 $psVersion = $psVersion.Major
 
 
  If ($psVersion -eq "3")
     {
     
         Function Get-Script
         {
             If ($myInvocation.ScriptName) { $myInvocation.ScriptName }
             else { $myInvocation.MyCommand.Definition }
         }
         
         $script = Get-Script
     
         Write-Host "PowerShell Version 3 Detected... Starting Powershell Version 2 Window"
         $powershell = "powershell.exe -version 2 -file $script"
         invoke-expression $powershell
         exit
     }
 
 <#
 Make-PS1ExeWrapper.ps1 Fuction
 Author: Keith Hill
 Date:   Aug 7, 2010
 http://rkeithhill.wordpress.com
 #>
 function make-exe
 {
 
 
 [CmdletBinding(DefaultParameterSetName="Path")]
 param(
     [Parameter(Mandatory=$true, Position=0, ParameterSetName="Path",
                ValueFromPipeline=$true, ValueFromPipelineByPropertyName=$true,
                HelpMessage="Path to bitmap file")]
     [ValidateNotNullOrEmpty()]
     [string[]]
     $Path,
     
     [Alias("PSPath")]
     [Parameter(Mandatory=$true, Position=0, ParameterSetName="LiteralPath", 
                ValueFromPipelineByPropertyName=$true,
                HelpMessage="Path to bitmap file")]
     [ValidateNotNullOrEmpty()]
     [string[]]
     $LiteralPath,
 
     [Parameter(Mandatory = $true, Position = 1)]
     [string]
     $OutputAssembly,
     
     [Parameter(Position = 2)]
     [string]
     $IconPath,
     
     [Parameter()]
     [switch]
     $STA
 )
 
 Begin {
     Set-StrictMode -Version latest 
     
     $MainAttribute = ''
     $ApartmentState = 'System.Threading.ApartmentState.MTA'
     if ($Sta)
     {
         $MainAttribute = '[STAThread]'
         $ApartmentState = 'System.Threading.ApartmentState.STA'
     }
     
     $src = @'
 using System;
 using System.Collections.Generic;
 using System.Collections.ObjectModel;
 using System.Globalization;
 using System.IO;
 using System.IO.Compression;
 using System.Management.Automation;
 using System.Management.Automation.Host;
 using System.Management.Automation.Runspaces;
 using System.Reflection;
 using System.Security;
 using System.Text;
 using System.Threading;
 
 namespace PS1ToExeTemplate
 {
     class Program
     {
         private static object _powerShellLock = new object();
         private static readonly Host _host = new Host();
         private static PowerShell _powerShellEngine;
 
 '@ + @"
 $MainAttribute
 
 "@ + @'
         static void Main(string[] args)
         {
             Console.CancelKeyPress += Console_CancelKeyPress;
             Console.TreatControlCAsInput = false;
 
             string script = GetScript();
             RunScript(script, args, null);
         }
 
         private static string GetScript()
         {
             string script = String.Empty;
 
             Assembly assembly = Assembly.GetExecutingAssembly();
             using (Stream stream = assembly.GetManifestResourceStream("Resources.Script.ps1.gz"))
             {
                 var gZipStream = new GZipStream(stream, CompressionMode.Decompress, true);
                 var streamReader = new StreamReader(gZipStream);
                 script = streamReader.ReadToEnd();
             }
 
             return script;
         }
 
         private static void RunScript(string script, string[] args, object input)
         {
             lock (_powerShellLock)
             {
                 _powerShellEngine = PowerShell.Create();
             }
 
             try
             {
                 _powerShellEngine.Runspace = RunspaceFactory.CreateRunspace(_host);
                 _powerShellEngine.Runspace.ApartmentState = 
 '@ + @"                
                 $ApartmentState;
                 
 "@ + @'                
                 _powerShellEngine.Runspace.Open();
                 _powerShellEngine.AddScript(script);
                 _powerShellEngine.AddCommand("Out-Default");
                 _powerShellEngine.Commands.Commands[0].MergeMyResults(PipelineResultTypes.Error, PipelineResultTypes.Output);
 
                 if (input != null)
                 {
                     _powerShellEngine.Invoke(new[] { input });
                 }
                 else
                 {
                     _powerShellEngine.Invoke();
                 }
             }
             finally
             {
                 lock (_powerShellLock)
                 {
                     _powerShellEngine.Dispose();
                     _powerShellEngine = null;
                 }
             }
         }
 
         private static void Console_CancelKeyPress(object sender, ConsoleCancelEventArgs e)
         {
             try
             {
                 lock (_powerShellLock)
                 {
                     if (_powerShellEngine != null && _powerShellEngine.InvocationStateInfo.State == PSInvocationState.Running)
                     {
                         _powerShellEngine.Stop();
                     }
                 }
                 e.Cancel = true;
             }
             catch (Exception ex)
             {
                 _host.UI.WriteErrorLine(ex.ToString());
             }
         }
     }
 
     class Host : PSHost
     {
         private PSHostUserInterface _psHostUserInterface = new HostUserInterface();
 
         public override void SetShouldExit(int exitCode)
         {
             Environment.Exit(exitCode);
         }
 
         public override void EnterNestedPrompt()
         {
             throw new NotImplementedException();
         }
 
         public override void ExitNestedPrompt()
         {
             throw new NotImplementedException();
         }
 
         public override void NotifyBeginApplication()
         {
         }
 
         public override void NotifyEndApplication()
         {
         }
 
         public override string Name
         {
             get { return "PSCX-PS1ToExeHost"; }
         }
 
         public override Version Version
         {
             get { return new Version(1, 0); }
         }
 
         public override Guid InstanceId
         {
             get { return new Guid("E4673B42-84B6-4C43-9589-95FAB8E00EB2"); }
         }
 
         public override PSHostUserInterface UI
         {
             get { return _psHostUserInterface; }
         }
 
         public override CultureInfo CurrentCulture
         {
             get { return Thread.CurrentThread.CurrentCulture; }
         }
 
         public override CultureInfo CurrentUICulture
         {
             get { return Thread.CurrentThread.CurrentUICulture; }
         }
     }
 
     class HostUserInterface : PSHostUserInterface, IHostUISupportsMultipleChoiceSelection
     {
         private PSHostRawUserInterface _psRawUserInterface = new HostRawUserInterface();
 
         public override PSHostRawUserInterface RawUI
         {
             get { return _psRawUserInterface; }
         }
 
         public override string ReadLine()
         {
             return Console.ReadLine();
         }
 
         public override SecureString ReadLineAsSecureString()
         {
             throw new NotImplementedException();
         }
 
         public override void Write(string value)
         {
             string output = value ?? "null";
             Console.Write(output);
         }
 
         public override void Write(ConsoleColor foregroundColor, ConsoleColor backgroundColor, string value)
         {
             string output = value ?? "null";
             var origFgColor = Console.ForegroundColor;
             var origBgColor = Console.BackgroundColor;
             Console.ForegroundColor = foregroundColor;
             Console.BackgroundColor = backgroundColor;
             Console.Write(output);
             Console.ForegroundColor = origFgColor;
             Console.BackgroundColor = origBgColor;
         }
 
         public override void WriteLine(string value)
         {
             string output = value ?? "null";
             Console.WriteLine(output);
         }
 
         public override void WriteErrorLine(string value)
         {
             string output = value ?? "null";
             var origFgColor = Console.ForegroundColor;
             Console.ForegroundColor = ConsoleColor.Red;
             Console.WriteLine(output);
             Console.ForegroundColor = origFgColor;
         }
 
         public override void WriteDebugLine(string message)
         {
             WriteYellowAnnotatedLine(message, "DEBUG");
         }
 
         public override void WriteVerboseLine(string message)
         {
             WriteYellowAnnotatedLine(message, "VERBOSE");
         }
 
         public override void WriteWarningLine(string message)
         {
             WriteYellowAnnotatedLine(message, "WARNING");
         }
 
         private void WriteYellowAnnotatedLine(string message, string annotation)
         {
             string output = message ?? "null";
             var origFgColor = Console.ForegroundColor;
             var origBgColor = Console.BackgroundColor;
             Console.ForegroundColor = ConsoleColor.Yellow;
             Console.BackgroundColor = ConsoleColor.Black;
             WriteLine(String.Format(CultureInfo.CurrentCulture, "{0}: {1}", annotation, output));
             Console.ForegroundColor = origFgColor;
             Console.BackgroundColor = origBgColor;
         }
 
         public override void WriteProgress(long sourceId, ProgressRecord record)
         {
             throw new NotImplementedException();
         }
 
         public override Dictionary<string, PSObject> Prompt(string caption, string message, Collection<FieldDescription> descriptions)
         {
             if (String.IsNullOrEmpty(caption) && String.IsNullOrEmpty(message) && descriptions.Count > 0)
             {
                 Console.Write(descriptions[0].Name + ": ");
             }
             else
             {
                 this.Write(ConsoleColor.DarkCyan, ConsoleColor.Black, caption + "\n" + message + " ");                
             }
             var results = new Dictionary<string, PSObject>();
             foreach (FieldDescription fd in descriptions)
             {
                 string[] label = GetHotkeyAndLabel(fd.Label);
                 this.WriteLine(label[1]);
                 string userData = Console.ReadLine();
                 if (userData == null)
                 {
                     return null;
                 }
 
                 results[fd.Name] = PSObject.AsPSObject(userData);
             }
 
             return results;
         }
 
         public override PSCredential PromptForCredential(string caption, string message, string userName, string targetName)
         {
             throw new NotImplementedException();
         }
 
         public override PSCredential PromptForCredential(string caption, string message, string userName, string targetName, PSCredentialTypes allowedCredentialTypes, PSCredentialUIOptions options)
         {
             throw new NotImplementedException();
         }
 
         public override int PromptForChoice(string caption, string message, Collection<ChoiceDescription> choices, int defaultChoice)
         {
             // Write the caption and message strings in Blue.
             this.WriteLine(ConsoleColor.Blue, ConsoleColor.Black, caption + "\n" + message + "\n");
 
             // Convert the choice collection into something that is
             // easier to work with. See the BuildHotkeysAndPlainLabels 
             // method for details.
             string[,] promptData = BuildHotkeysAndPlainLabels(choices);
 
             // Format the overall choice prompt string to display.
             var sb = new StringBuilder();
             for (int element = 0; element < choices.Count; element++)
             {
                 sb.Append(String.Format(CultureInfo.CurrentCulture, "|{0}> {1} ", promptData[0, element], promptData[1, element]));
             }
 
             sb.Append(String.Format(CultureInfo.CurrentCulture, "[Default is ({0}]", promptData[0, defaultChoice]));
 
             // Read prompts until a match is made, the default is
             // chosen, or the loop is interrupted with ctrl-C.
             while (true)
             {
                 this.WriteLine(sb.ToString());
                 string data = Console.ReadLine().Trim().ToUpper(CultureInfo.CurrentCulture);
 
                 // If the choice string was empty, use the default selection.
                 if (data.Length == 0)
                 {
                     return defaultChoice;
                 }
 
                 // See if the selection matched and return the
                 // corresponding index if it did.
                 for (int i = 0; i < choices.Count; i++)
                 {
                     if (promptData[0, i] == data)
                     {
                         return i;
                     }
                 }
 
                 this.WriteErrorLine("Invalid choice: " + data);
             }
         }
 
 
         public Collection<int> PromptForChoice(string caption, string message, Collection<ChoiceDescription> choices, IEnumerable<int> defaultChoices)
         {
             this.WriteLine(ConsoleColor.Blue, ConsoleColor.Black, caption + "\n" + message + "\n");
 
             string[,] promptData = BuildHotkeysAndPlainLabels(choices);
 
             var sb = new StringBuilder();
             for (int element = 0; element < choices.Count; element++)
             {
                 sb.Append(String.Format(CultureInfo.CurrentCulture, "|{0}> {1} ", promptData[0, element], promptData[1, element]));
             }
 
             var defaultResults = new Collection<int>();
             if (defaultChoices != null)
             {
                 int countDefaults = 0;
                 foreach (int defaultChoice in defaultChoices)
                 {
                     ++countDefaults;
                     defaultResults.Add(defaultChoice);
                 }
 
                 if (countDefaults != 0)
                 {
                     sb.Append(countDefaults == 1 ? "[Default choice is " : "[Default choices are ");
                     foreach (int defaultChoice in defaultChoices)
                     {
                         sb.AppendFormat(CultureInfo.CurrentCulture, "\"{0}\",", promptData[0, defaultChoice]);
                     }
                     sb.Remove(sb.Length - 1, 1);
                     sb.Append("]");
                 }
             }
 
             this.WriteLine(ConsoleColor.Cyan, ConsoleColor.Black, sb.ToString());
 
             var results = new Collection<int>();
             while (true)
             {
             ReadNext:
                 string prompt = string.Format(CultureInfo.CurrentCulture, "Choice[{0}]:", results.Count);
                 this.Write(ConsoleColor.Cyan, ConsoleColor.Black, prompt);
                 string data = Console.ReadLine().Trim().ToUpper(CultureInfo.CurrentCulture);
 
                 if (data.Length == 0)
                 {
                     return (results.Count == 0) ? defaultResults : results;
                 }
 
                 for (int i = 0; i < choices.Count; i++)
                 {
                     if (promptData[0, i] == data)
                     {
                         results.Add(i);
                         goto ReadNext;
                     }
                 }
 
                 this.WriteErrorLine("Invalid choice: " + data);
             }
         }
 
 
         private static string[,] BuildHotkeysAndPlainLabels(Collection<ChoiceDescription> choices)
         {
             // Allocate the result array
             string[,] hotkeysAndPlainLabels = new string[2, choices.Count];
 
             for (int i = 0; i < choices.Count; ++i)
             {
                 string[] hotkeyAndLabel = GetHotkeyAndLabel(choices[i].Label);
                 hotkeysAndPlainLabels[0, i] = hotkeyAndLabel[0];
                 hotkeysAndPlainLabels[1, i] = hotkeyAndLabel[1];
             }
 
             return hotkeysAndPlainLabels;
         }
 
         private static string[] GetHotkeyAndLabel(string input)
         {
             string[] result = new string[] { String.Empty, String.Empty };
             string[] fragments = input.Split('&');
             if (fragments.Length == 2)
             {
                 if (fragments[1].Length > 0)
                 {
                     result[0] = fragments[1][0].ToString().
                     ToUpper(CultureInfo.CurrentCulture);
                 }
 
                 result[1] = (fragments[0] + fragments[1]).Trim();
             }
             else
             {
                 result[1] = input;
             }
 
             return result;
         }
     }
 
     class HostRawUserInterface : PSHostRawUserInterface
     {
         public override KeyInfo ReadKey(ReadKeyOptions options)
         {
             throw new NotImplementedException();
         }
 
         public override void FlushInputBuffer()
         {
         }
 
         public override void SetBufferContents(Coordinates origin, BufferCell[,] contents)
         {
             throw new NotImplementedException();
         }
 
         public override void SetBufferContents(Rectangle rectangle, BufferCell fill)
         {
             throw new NotImplementedException();
         }
 
         public override BufferCell[,] GetBufferContents(Rectangle rectangle)
         {
             throw new NotImplementedException();
         }
 
         public override void ScrollBufferContents(Rectangle source, Coordinates destination, Rectangle clip, BufferCell fill)
         {
             throw new NotImplementedException();
         }
 
         public override ConsoleColor ForegroundColor
         {
             get { return Console.ForegroundColor; }
             set { Console.ForegroundColor = value; }
         }
 
         public override ConsoleColor BackgroundColor
         {
             get { return Console.BackgroundColor; }
             set { Console.BackgroundColor = value; }
         }
 
         public override Coordinates CursorPosition
         {
             get { return new Coordinates(Console.CursorLeft, Console.CursorTop); }
             set { Console.SetCursorPosition(value.X, value.Y); }
         }
 
         public override Coordinates WindowPosition
         {
             get { return new Coordinates(Console.WindowLeft, Console.WindowTop); }
             set { Console.SetWindowPosition(value.X, value.Y); }
         }
 
         public override int CursorSize
         {
             get { return Console.CursorSize; }
             set { Console.CursorSize = value; }
         }
 
         public override Size BufferSize
         {
             get { return new Size(Console.BufferWidth, Console.BufferHeight); }
             set { Console.SetBufferSize(value.Width, value.Height); }
         }
 
         public override Size WindowSize
         {
             get { return new Size(Console.WindowWidth, Console.WindowHeight); }
             set { Console.SetWindowSize(value.Width, value.Height); }
         }
 
         public override Size MaxWindowSize
         {
             get { return new Size(Console.LargestWindowWidth, Console.LargestWindowHeight); }
         }
 
         public override Size MaxPhysicalWindowSize
         {
             get { return new Size(Console.LargestWindowWidth, Console.LargestWindowHeight); }
         }
 
         public override bool KeyAvailable
         {
             get { return Console.KeyAvailable; }
         }
 
         public override string WindowTitle
         {
             get { return Console.Title; }
             set { Console.Title = value; }
         }
     }
 }
 '@
 }    
 
 Process {
     if ($psCmdlet.ParameterSetName -eq "Path")
     {
         $resolvedPaths = @($Path | Resolve-Path | Convert-Path)
     }
     else 
     {
         $resolvedPaths = @($LiteralPath | Convert-Path)
     }
  
     foreach ($rpath in $resolvedPaths) 
     {
         Write-Verbose "Processing $rpath"
 
         $gzItem = Get-ChildItem $rpath | Write-GZip -Quiet 
         $resourcePath = "$($gzItem.Directory)\Resources.Script.ps1.gz"
         if (Test-Path $resourcePath) { Remove-Item $resourcePath }
         Rename-Item $gzItem $resourcePath
         
         $referenceAssemblies = 'System.dll',([psobject].Assembly.Location)
         $outputPath = $OutputAssembly
         if (![IO.Path]::IsPathRooted($outputPath))
         {
             $outputPath = [io.path]::GetFullPath((Join-Path $pwd $outputPath))
         }
         if ($rpath -eq $outputPath)
         { 
             throw 'Oops, you don''t really want to overwrite your script with an EXE.' 
         }
 
         $cp = new-object System.CodeDom.Compiler.CompilerParameters $referenceAssemblies,$outputPath,$true
         $cp.TempFiles = new-object System.CodeDom.Compiler.TempFileCollection ([IO.Path]::GetTempPath())
         $cp.GenerateExecutable = $true
         $cp.GenerateInMemory   = $false
         $cp.IncludeDebugInformation = $true
         if ($IconPath) 
         {
             $rIconPath = Resolve-Path $IconPath
             $cp.CompilerOptions = " /win32icon:$rIconPath"
         }
         [void]$cp.EmbeddedResources.Add($resourcePath)
         
         $dict = new-object 'System.Collections.Generic.Dictionary[string,string]' 
         $dict.Add('CompilerVersion','v3.5')
         $provider = new-object Microsoft.CSharp.CSharpCodeProvider $dict
         
         $results = $provider.CompileAssemblyFromSource($cp, $src)
         if ($results.Errors.Count)
         {
             $errorLines = "" 
             foreach ($error in $results.Errors) 
             { 
                 $errorLines += "`n`t" + $error.Line + ":`t" + $error.ErrorText 
             }
             Write-Error $errorLines
         }
     }  
 }}
 
 
     function Reload-Profile {
     @(
         $Profile.AllUsersAllHosts,
         $Profile.AllUsersCurrentHost,
         $Profile.CurrentUserAllHosts,
         $Profile.CurrentUserCurrentHost
     ) | % {
         if(Test-Path $_){
             Write-Verbose "Running $_"
             . $_
         }
     }    
 }
 
 
 $pspCheck = $NULL
 $pspCheck = Get-Module -ListAvailable | Where-Object {$_.Name -eq "PowerShellPack"}
 if (!$pspCheck)
 {
     Write-Host "Installing PowerShellPack... Please Wait..."
     $install = "C:\Tools\PowerShellPack.msi /quiet"
     invoke-expression $install
     Start-Sleep -Seconds 30
     Reload-Profile
         
 }
 $pscxCheck = $NULL
 $pscxCheck = Get-Module -ListAvailable | Where-Object {$_.Name -eq "Pscx"}
 $psVer = ($host.version).Major
 
 
 
 if (!$pscxCheck)
 {
     Write-Host "Installing PSCX..." 
     Copy-Item C:\Tools\Pscx C:\Windows\System32\WindowsPowerShell\v1.0\Modules -Recurse
   
     Start-Sleep -Seconds 30
     Reload-Profile
         
 }
 
 Import-Module PowerShellPack
 Import-Module Pscx
 
 		$xApp = ""
         $xTask = ""
         $xProj = ""
         $xSA = ""
 
 function GenerateForm {
 ########################################################################
 ########################################################################
 
 [reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
 [reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 
 $form = New-Object System.Windows.Forms.Form
 $exitBtn = New-Object System.Windows.Forms.Button
 $okBtn = New-Object System.Windows.Forms.Button
 $fileBtn = New-Object System.Windows.Forms.Button
 $cleanupCkbx = New-Object System.Windows.Forms.CheckBox
 $passTxt = New-Object System.Windows.Forms.TextBox
 $passLbl = New-Object System.Windows.Forms.Label
 $svcAcctTxt = New-Object System.Windows.Forms.ComboBox
 $svcAcctLbl = New-Object System.Windows.Forms.Label
 $projShortTxt = New-Object System.Windows.Forms.TextBox
 $projShortLbl = New-Object System.Windows.Forms.Label
 $taskNameTxt = New-Object System.Windows.Forms.TextBox
 $taskNameLbl = New-Object System.Windows.Forms.Label
 $appPathTxt = New-Object System.Windows.Forms.TextBox
 $appPathLbl = New-Object System.Windows.Forms.Label
 $progressBar = New-Object System.Windows.Forms.ProgressBar
 $timer1 = New-Object System.Windows.Forms.Timer
 $timer2 = New-Object System.Windows.Forms.Timer
 $openFileDialog1 = New-Object System.Windows.Forms.OpenFileDialog
 $InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
 #----------------------------------------------
 #----------------------------------------------
 
 function stepBar
 {
     if ($progressBar.Value -lt 100)
     {    
     $progressBar.Increment(1)
     }
     else
     {
     $progressBar.Value = 1
     }
 }
 
 $okBtn_OnClick= 
 {
 
 $timer1.Interval = 1000
 $timer1.add_Tick({stepBar})
 $timer1.start()
 
 $xApp=$appPathTxt.Text
 $xTask=$taskNameTxt.Text
 $xProj=$projShortTxt.Text
 $xSA=$svcAcctTxt.Text
 $xPass=$passTxt.Text
 
 
 $newDir = "C:\Tools\"+$xTask
 New-Item $newDir -type directory
 $scriptRun = $newDir+"\"+$xTask + ".ps1"
 $scriptTask = $newDir+"\"+$xTask + "_Task.ps1"
 $scriptTaskExe = $newDir+"\"+$xTask + "_Task.exe"
 $scriptExe = $newDir+"\"+$xTask + ".exe"
 New-Item $scriptRun -type file -force
 New-Item $scriptTask -type file -force
 if ($winVersion -like "*2003*")
 {
    $taskDir = "C:\WINDOWS\Tasks\"+$xProj
 }
 else
 {
 $taskDir = "C:\Windows\System32\Tasks\"+$xProj
 }
 New-Item $taskDir -type directory -force
 
 
 #$netuser = $xSA+" "+$password+" /add"
 Write-Host "Creating Local User"
 net user $xSA $xPass /add
 net localgroup administrators /add $xSA
 
 $localGroup = $xProj+"_TaskRunners"
 
 Write-Host "Creating Local Group"
 net localgroup $localGroup /add
 
 Write-Host "Setting Permissions"
 $cacls = "cacls.exe "+$taskDir+" /T /E /G "+$localGroup+":R"
 Invoke-Expression $cacls
 
 Write-Host "Copying Module"
 
 New-Item C:\Windows\System32\WindowsPowerShell\v1.0\Modules\PSTerminalServices -type directory -Force
 Copy-Item C:\Tools\PSTerminalServices\* C:\Windows\System32\WindowsPowerShell\v1.0\Modules\PSTerminalServices\ -Recurse -ErrorAction SilentlyContinue
 if ($winVersion -like "*2003*")
 {
 $Task = $xTask
 $newTask = $scriptTaskExe
 }
 else
 {
 $Task = $xProj+"\"+$xTask
 $newTask = $scriptTaskExe
 }
 
 if ($winVersion -like "*2003*")
 {
 SCHTASKS /CREATE /RU $xSA /RP $xPass /TN $Task /TR $newTask /SC ONCE /ST 00:01 /F
 }
 else
 {
 New-Task | Add-TaskAction -Path $newTask -WorkingDirectory "" | Register-ScheduledTask $Task 
 SCHTASKS /Change /RU $xSA /RP $xPass /TN $Task /RL HIGHEST
 }
 
 $tempPath = "C:\Tools\"+$xTask+"\Temp"
 New-Item -Type Directory -Path $tempPath
 
 $scriptRunContents = @"
 Import-Module PSTerminalServices
 `$tempFile`="C:\Tools\$xTask\Temp\session.tmp"
 `$logFile`="C:\Tools\$xTask\Temp\log.txt"
 `$task` = "$Task"
 `$user`=whoami
 `$user`=`$user`.Split("\")[1]
 
 `$getSessions`=Get-TSSession
 `$getSessions`=`$getSessions`|Select "SessionID", "UserName"
 
 `$userSession`=`$getSessions`|Where-Object {`$_.UserName` -eq `$user`}
 `$sessionId`=`$userSession.SessionId`
 
 `$sessionId`|Out-File `$tempFile`
 
 
 schtasks /run /TN $task
 "@
 Out-File -InputObject $scriptRunContents -FilePath $scriptRun
 $cacls = "cacls.exe "+$scriptRun+" /T /E /G "+$localGroup+":R"
 Invoke-Expression $cacls
 
 $cacls = "cacls.exe "+$tempPath+" /T /E /G "+$localGroup+":C"
 Invoke-Expression $cacls
 
 Write-Host "Copying PsExec to System32"
 Copy-Item -Path C:\Tools\PsExec.exe C:\Windows\System32\
 
 $scriptTaskContents = @"
 Import-Module PSTerminalServices
 `$logFile`="C:\Tools\$xTask\Temp\log.txt"
 `$tempFile`="C:\Tools\$xTask\Temp\session.tmp"
 `$psexec`="PsExec.exe /accepteula"
 `$command`=`"cmd /C ```"$xApp```"`"
 `$session`=get-content `$tempFile`
 
 `$cmd`=`$psexec`+" -s -d -i "+`$session`+" "+`$command`
 
 Get-Date|Out-File `$logFile` -Append
 `$event` = Get-TSSession -SessionID `$session`|Select "SessionID", "IPAddress", "UserName"|Format-Table -autosize
 
 Invoke-Expression `$cmd`
 
 Remove-Item `$tempFile`
 "@
 Out-File -InputObject $scriptTaskContents -FilePath $scriptTask
 
 make-exe $scriptRun $scriptExe C:\Tools\Run.ico
 make-exe $scriptTask $scriptTaskExe C:\Tools\Run.ico
 $pdbFile = $newDir+"\*.pdb"
 $gzFile = $newDir+"\*.gz"
 Remove-Item $pdbFile
 Remove-Item $gzFile
 Remove-Item $scriptRun
 Remove-Item $scriptTask
 
 $chkbox = $cleanupCkbx.Checked
 
 if ($chkbox -eq $TRUE)
 {
     $PscxFiles = "C:\Tools\Pscx"
     Remove-Item $PscxFiles -Recurse -Force
     $PSTSFiles = "C:\Tools\PSTerminalServices"
     Remove-Item $PSTSFiles -Recurse -Force
     Remove-Item C:\Tools\PowerShellPack.msi -Force
     Remove-Item C:\Tools\Run.ico
     Remove-Item C:\Tools\PsExec.exe
 }
 
 
 $xPass = $NULL
 $timer1.Stop()
 $form.Close()
 
 }
 
 $handler_form_Load= 
 {
 $progressBar.Value = 0
 $timer1.Stop()
 }
 
 $exitBtn_OnClick= 
 {
 $timer1.Stop()
 $form.Close()
 
 }
 
 $fileBtn_OnClick= 
 {
 $openFileDialog1.ShowDialog()
 $appPathTxt.Text = $openFileDialog1.FileName
 
 }
 
 
 
 $OnLoadForm_StateCorrection=
 	$form.WindowState = $InitialFormWindowState
 }
 $users = get-wmiobject -class "Win32_UserAccount" -Filter "LocalAccount = True"
 
 foreach ($user in $users)
 {
     $svcAcctTxt.Items.Add($user.Name)
 }
 #----------------------------------------------
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 268
 $System_Drawing_Size.Width = 229
 $form.ClientSize = $System_Drawing_Size
 $form.DataBindings.DefaultDataSourceUpdateMode = 0
 $form.Name = "form"
 $form.Text = "Deploy RunAsAdmin"
 $form.add_Load($handler_form_Load)
 
 
 $exitBtn.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 119
 $System_Drawing_Point.Y = 223
 $exitBtn.Location = $System_Drawing_Point
 $exitBtn.Name = "exitBtn"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 98
 $exitBtn.Size = $System_Drawing_Size
 $exitBtn.TabIndex = 14
 $exitBtn.Text = "EXIT"
 $exitBtn.UseVisualStyleBackColor = $True
 $exitBtn.add_Click($exitBtn_OnClick)
 
 $form.Controls.Add($exitBtn)
 
 
 $okBtn.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 12
 $System_Drawing_Point.Y = 223
 $okBtn.Location = $System_Drawing_Point
 $okBtn.Name = "okBtn"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 98
 $okBtn.Size = $System_Drawing_Size
 $okBtn.TabIndex = 13
 $okBtn.Text = "OK"
 $okBtn.UseVisualStyleBackColor = $True
 $okBtn.add_Click($okBtn_OnClick)
 
 $form.Controls.Add($okBtn)
 
 
 $fileBtn.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 191
 $System_Drawing_Point.Y = 19
 $fileBtn.Location = $System_Drawing_Point
 $fileBtn.Name = "fileBtn"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 26
 $fileBtn.Size = $System_Drawing_Size
 $fileBtn.TabIndex = 12
 $fileBtn.Text = "..."
 $fileBtn.UseVisualStyleBackColor = $True
 $fileBtn.add_Click($fileBtn_OnClick)
 
 $form.Controls.Add($fileBtn)
 
 
 $cleanupCkbx.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 12
 $System_Drawing_Point.Y = 200
 $cleanupCkbx.Location = $System_Drawing_Point
 $cleanupCkbx.Name = "cleanupCkbx"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 16
 $System_Drawing_Size.Width = 300
 $cleanupCkbx.Size = $System_Drawing_Size
 $cleanupCkbx.TabIndex = 11
 $cleanupCkbx.Text = "Cleanup source files after completion "
 $cleanupCkbx.UseVisualStyleBackColor = $True
 $cleanupCkbx.add_CheckedChanged($handler_cleanupCkbx_CheckedChanged)
 
 $form.Controls.Add($cleanupCkbx)
 
 $passTxt.DataBindings.DefaultDataSourceUpdateMode = 0
 $passTxt.HideSelection = $False
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 12
 $System_Drawing_Point.Y = 176
 $passTxt.Location = $System_Drawing_Point
 $passTxt.Name = "passTxt"
 $passTxt.PasswordChar = '*'
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 205
 $passTxt.Size = $System_Drawing_Size
 $passTxt.TabIndex = 10
 
 $form.Controls.Add($passTxt)
 
 $passLbl.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 12
 $System_Drawing_Point.Y = 160
 $passLbl.Location = $System_Drawing_Point
 $passLbl.Name = "passLbl"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 12
 $System_Drawing_Size.Width = 205
 $passLbl.Size = $System_Drawing_Size
 $passLbl.TabIndex = 9
 $passLbl.Text = "Service Account Password:"
 
 $form.Controls.Add($passLbl)
 
 $svcAcctTxt.DataBindings.DefaultDataSourceUpdateMode = 0
 $svcAcctTxt.FormattingEnabled = $True
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 12
 $System_Drawing_Point.Y = 136
 $svcAcctTxt.Location = $System_Drawing_Point
 $svcAcctTxt.Name = "svcAcctTxt"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 21
 $System_Drawing_Size.Width = 205
 $svcAcctTxt.Size = $System_Drawing_Size
 $svcAcctTxt.TabIndex = 8
 
 $form.Controls.Add($svcAcctTxt)
 
 $svcAcctLbl.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 12
 $System_Drawing_Point.Y = 120
 $svcAcctLbl.Location = $System_Drawing_Point
 $svcAcctLbl.Name = "svcAcctLbl"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 12
 $System_Drawing_Size.Width = 205
 $svcAcctLbl.Size = $System_Drawing_Size
 $svcAcctLbl.TabIndex = 7
 $svcAcctLbl.Text = "Service Account Name:"
 
 $form.Controls.Add($svcAcctLbl)
 
 $projShortTxt.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 97
 $projShortTxt.Location = $System_Drawing_Point
 $projShortTxt.Name = "projShortTxt"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 204
 $projShortTxt.Size = $System_Drawing_Size
 $projShortTxt.TabIndex = 6
 
 $form.Controls.Add($projShortTxt)
 
 $projShortLbl.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 81
 $projShortLbl.Location = $System_Drawing_Point
 $projShortLbl.Name = "projShortLbl"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 12
 $System_Drawing_Size.Width = 204
 $projShortLbl.Size = $System_Drawing_Size
 $projShortLbl.TabIndex = 5
 $projShortLbl.Text = "Project Short-Name:"
 
 $form.Controls.Add($projShortLbl)
 
 $taskNameTxt.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 58
 $taskNameTxt.Location = $System_Drawing_Point
 $taskNameTxt.Name = "taskNameTxt"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 204
 $taskNameTxt.Size = $System_Drawing_Size
 $taskNameTxt.TabIndex = 4
 
 $form.Controls.Add($taskNameTxt)
 
 $taskNameLbl.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 42
 $taskNameLbl.Location = $System_Drawing_Point
 $taskNameLbl.Name = "taskNameLbl"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 12
 $System_Drawing_Size.Width = 204
 $taskNameLbl.Size = $System_Drawing_Size
 $taskNameLbl.TabIndex = 3
 $taskNameLbl.Text = "Task Name:"
 
 $form.Controls.Add($taskNameLbl)
 
 $appPathTxt.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 19
 $appPathTxt.Location = $System_Drawing_Point
 $appPathTxt.Name = "appPathTxt"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 172
 $appPathTxt.Size = $System_Drawing_Size
 $appPathTxt.TabIndex = 2
 
 $form.Controls.Add($appPathTxt)
 
 $appPathLbl.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 12
 $System_Drawing_Point.Y = 3
 $appPathLbl.Location = $System_Drawing_Point
 $appPathLbl.Name = "appPathLbl"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 12
 $System_Drawing_Size.Width = 205
 $appPathLbl.Size = $System_Drawing_Size
 $appPathLbl.TabIndex = 1
 $appPathLbl.Text = "Application Path:"
 
 $form.Controls.Add($appPathLbl)
 
 $progressBar.DataBindings.DefaultDataSourceUpdateMode = 0
 $progressBar.ForeColor = [System.Drawing.Color]::FromArgb(255,160,160,160)
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 252
 $progressBar.Location = $System_Drawing_Point
 $progressBar.Name = "progressBar"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 10
 $System_Drawing_Size.Width = 205
 $progressBar.Size = $System_Drawing_Size
 $progressBar.TabIndex = 0
 
 $form.Controls.Add($progressBar)
 
 
 
 $openFileDialog1.FileName = "openFileDialog1"
 $openFileDialog1.ShowHelp = $True
 
 
 $InitialFormWindowState = $form.WindowState
 $form.add_Load($OnLoadForm_StateCorrection)
 $form.ShowDialog()| Out-Null
 
 
 GenerateForm
`

