---
Author: cz9qvh
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 1043
Published Date: 
Archived Date: 2009-06-05t16
---

# pslauncher - 

## Description

requires v2

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 /*
  * Compile this program by simply doing:
  * 
  * csc.exe PSLauncher.cs
  * 
  */
 
 // Requires powershell V2
 
 /*
  * The Problem:
  *      .ps1 scripts do not integrate with the shell nicely.  You cannot
  *      simply associate powershell.exe with .ps1 and have them run when u
  *      double click them.  Also, the execution of script you click on may
  *      be blocked by security policies.  Thirdly, some powershell scripts
  *      require ps to be launched with some command line arguments, to frequently
  *      run such a script then requires keyboard entry or awkward batch files.
  * 
  * The Solution:
  *      Compile this program, and associate it as the default launcher for .ps1.
  *      follow the usage details below, and you will be able to launch .ps1 scripts
  *      from the shell with mouse alone, and as added benafit you get to put
  *      ps cmd line options in your script, and you can run scripts invisible.
  * 
  * USAGE:
  *      Put the first two characters of your script as "#!"
  *      Follow that with the full path to powershell.exe (incl fname) or
  *      if powershell.exe is in the path you can just put #!powershell.exe
  * 
  *      The remainder part of that line will be passed to powershell.exe as
  *      arguments.  For example, -STA.  The filename of the script is also
  *      passed as "-file %1" along with "-ExecutionPolicy UnRestricted".
  * 
  *      The second line of the file, if it is prefixed by "#-" can contain "meta arguments"
  *      for powershell.  Currently the only one allowed is "Hidden" which will supress the
  *      powershell window and discard any output from the script.
  * 
  *      If a .ps1 script without these magics #! is encountered, it will be passed
  *      to powershell.exe as above, if powershell.exe is in the path or default location.
  * 
  * Example:
  *      #!powershell.exe -STA
  *      #-Hidden
  *      ...rest of script...
  * 
  * 
  *  or
  * 
  * 
  *      #!C:\WINDOWS\system32\windowspowershell\v1.0\powershell.exe -noexit
  *      ...rest of a script you don't want to exit when it ends...
  */
 
 using System;
 using System.Collections.Generic;
 using System.Windows.Forms;
 using System.Diagnostics;
 using System.IO;
 using System.Text.RegularExpressions;
 
 namespace PSLauncher
 {
     static class PSLauncher
     {
         /// <summary>
         /// The main entry point for the application.
         /// </summary>
         [STAThread]
         static void Main(string [] args)
         {
             if (args.Length != 1)
             {
                 throw new ArgumentException("Wrong number of arguments, there must be exactly one.");
             }
             if (!(new Regex(".ps1$")).Match(args[0]).Success)
             {
                 psl_err("Must have extension of ps1 or powershell.exe will error.");
                 return;
             }
             StreamReader fs;
             string line_1;
             string line_2;
             string exe_path;
             string cmd_line;
             string ps_args;
             Regex r;
             Match m;
             bool hidden = false;
             Process ps_proc;
 
             try
             {
                 fs = new StreamReader(args[0]);
             }
             catch (FileNotFoundException)
             {
                 psl_err("File Not Found.");
                 return;
             }
             catch (FileLoadException)
             {
                 psl_err("File found but could not be loaded.");
                 return;
             }
             try
             {
                 line_1 = fs.ReadLine();
                 line_2 = fs.ReadLine();
             }
             catch (IOException)
             {
                 psl_err("File may not have enough lines.");
                 return;
             }
             fs.Close();
             r        = new Regex(@"^#!(.*)powershell\.exe");
             exe_path = "";
             if ((m = r.Match(line_1)).Success) 
             {
                 line_1   = r.Replace(line_1, "");
                 exe_path = m.Groups[1].Value;
                 if (exe_path  != "")
                 {
                     if (!File.Exists(exe_path + "powershell.exe"))
                     {
                         psl_err("specified powershell.exe file not exist.");
                     }
                 }
                 else
                 {
                     exe_path = find_powershell();
                     if (exe_path == "")
                     {
                         psl_err("powershell.exe is not found.");
                         return;
             }   }   }
             else
             {
                 line_1   = "";
                 exe_path = find_powershell();
                 if (exe_path == "")
                 {
                     psl_err("powershell.exe is not found.");
                     return;
             }   }
             cmd_line = exe_path + "\\" + "powershell.exe";
             ps_args  = line_1 + " -ExecutionPolicy UnRestricted -File \"" + args[0] + "\"";
             r        = new Regex("^#-");
             if ((m = r.Match(line_2)).Success)
             {
                 line_2 = r.Replace(line_2, "");
                 if ((new Regex("Hidden")).Match(line_2).Success)
                 {
                     hidden = true;
             }   }
             ps_proc = new Process();
 
             if (hidden)
             {
                 ps_proc.StartInfo.RedirectStandardError  = true;
                 ps_proc.StartInfo.RedirectStandardInput  = true;
                 ps_proc.StartInfo.RedirectStandardOutput = true;
                 ps_proc.StartInfo.CreateNoWindow = true;
                 ps_proc.StartInfo.UseShellExecute = false;
             }
             ps_proc.StartInfo.Arguments = ps_args;
             ps_proc.StartInfo.FileName = cmd_line;
             ps_proc.Start();
         }
         private static void psl_err(string msg)
         {
             MessageBox.Show(msg, "PSLauncher Critical Error", MessageBoxButtons.OK, 
                 MessageBoxIcon.Error);
         }
         private static string find_powershell()
         {
             string exe_path = "";
             foreach (string test_path in
                         Environment.GetEnvironmentVariable("Path").Split(';'))
             {
                 if (File.Exists(test_path + "\\" + "powershell.exe"))
                     exe_path = test_path;
             }
             if (exe_path == "" && File.Exists(
                 "C:\\WINDOWS\\system32\\windowspowershell\\v1.0\\powershell.exe"))
                 exe_path = "C:\\WINDOWS\\system32\\windowspowershell\\v1.0\\powershell.exe";
 
             return exe_path;
 }   }   }
`

