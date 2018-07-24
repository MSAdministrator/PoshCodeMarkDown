---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc-by-sa-nc
PoshCode ID: 2008
Published Date: 2012-07-22t20
Archived Date: 2012-11-30t21
---

# desktop - 

## Description

a c# class (use with add-type -path) that encapsulates the desktop apis…

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 // Desktop 1.1
 //																																						*
 // Copyright (C) 2004  http://www.onyeyiri.co.uk
 // Coded by: Nnamdi Onyeyiri
 //
 // This code may be used in any way you desire except for commercial use.
 // The code can be freely redistributed unmodified as long as all of the authors 
 // copyright notices remain intact, as long as it is not being sold for profit.
 // Although I have tried to ensure the stability of the code, I will not take 
 // responsibility for any loss, damages, or death it may cause.
 //
 // This software is provided "as is" without any expressed or implied warranty.
 //
 // -------------------------
 // Change Log
 //
 // Version 1.0:  -First Release
 //
 // Version 1.1:  -Added Window and WindowCollection classes
 // 6/6/2004      -Added another GetWindows overload, that used WindowCollection
 //               -Added GetInputProcesses method to retrieve processes on Input desktop
 //               -Changed GetWindows and GetDesktops to return arrays, instead of them being passed by ref
 //
 // Version 1.2   -Implemented IDisposable
 // 8/7/2004      -Implemented ICloneable
 //               -Overrided ToString to return desktop name
 //
 // Version 2.0   -Switched to Generic Collections
 // 7/19/2010     -Added enumeration of process from alternate desktops
 // Joel Bennett  -Renamed a lot of methods to remove duplication
 
 using System;
 using System.Threading;
 using System.Text;
 using System.Diagnostics;
 using System.Runtime.InteropServices;
 using System.Collections.Generic;
 using System.ComponentModel;
 
 /// <summary>
 /// Encapsulates the Desktop API.
 /// </summary>
 public class Desktop : IDisposable, ICloneable
 {
    [DllImport("kernel32.dll")]
    private static extern int GetThreadId(IntPtr thread);
 
    [DllImport("kernel32.dll")]
    private static extern int GetProcessId(IntPtr process);
 
    //
    // Imported winAPI functions.
    //
    [DllImport("user32.dll")]
    private static extern IntPtr CreateDesktop(string lpszDesktop, IntPtr lpszDevice, IntPtr pDevmode, int dwFlags, long dwDesiredAccess, IntPtr lpsa); 
 
    [DllImport("user32.dll")]
    private static extern bool CloseDesktop(IntPtr hDesktop);
 
    [DllImport("user32.dll")]
    private static extern IntPtr OpenDesktop(string lpszDesktop, int dwFlags, bool fInherit, long dwDesiredAccess);
 
    [DllImport("user32.dll")]
    private static extern IntPtr OpenInputDesktop(int dwFlags, bool fInherit, long dwDesiredAccess);
 
    [DllImport("user32.dll")]
    private static extern bool SwitchDesktop(IntPtr hDesktop);
 
    [DllImport("user32.dll")]
    private static extern bool EnumDesktops(IntPtr hwinsta, EnumDesktopProc lpEnumFunc, IntPtr lParam);
 
    [DllImport("user32.dll")]
    private static extern IntPtr GetProcessWindowStation();
 
    [DllImport("user32.dll")]
    private static extern bool EnumDesktopWindows(IntPtr hDesktop, EnumDesktopWindowsProc lpfn, IntPtr lParam);
 
    [DllImport("user32.dll")]
    private static extern bool SetThreadDesktop(IntPtr hDesktop);
 
    [DllImport("user32.dll")]
    public static extern IntPtr GetThreadDesktop(int dwThreadId);
 
    [DllImport("user32.dll")]
    [return: MarshalAs(UnmanagedType.Bool)]
    private static extern bool GetUserObjectInformation(IntPtr hObj, int nIndex, IntPtr pvInfo, int nLength, ref int lpnLengthNeeded);
 
    
    
    [DllImport("user32.dll")]
    private static extern int GetWindowText(IntPtr hWnd, IntPtr lpString, int nMaxCount);
 
    private delegate bool EnumDesktopProc(string lpszDesktop, IntPtr lParam);
    private delegate bool EnumDesktopWindowsProc(IntPtr desktopHandle, IntPtr lParam);
 
    
    // The Unicode version of this function, CreateProcessW, can modify the contents of the lpCommandLine string.
    // Therefore, you can't just pass a string (because it's a constant, so it becomes a pointer to read-only memory).
    [return: MarshalAs(UnmanagedType.Bool)][DllImport("kernel32.dll")]
    private static extern bool CreateProcess( string lpApplicationName, string lpCommandLine, IntPtr lpProcessAttributes, IntPtr lpThreadAttributes, bool bInheritHandles, int dwCreationFlags,  IntPtr lpEnvironment,  string lpCurrentDirectory,  [In] ref STARTUPINFO lpStartupInfo, [Out]out PROCESS_INFORMATION lpProcessInformation);
 
    [StructLayout(LayoutKind.Sequential)]
    private struct PROCESS_INFORMATION 
    {
       public IntPtr hProcess;
       public IntPtr hThread;
       public int dwProcessId;
       public int dwThreadId;
    }
 
    [StructLayout(LayoutKind.Sequential)]
    private struct STARTUPINFO 
    {
       public int cb;
       public string lpReserved;
       public string lpDesktop;
       public string lpTitle;
       public int dwX;
       public int dwY;
       public int dwXSize;
       public int dwYSize;
       public int dwXCountChars;
       public int dwYCountChars;
       public int dwFillAttribute;
       public int dwFlags;
       public short wShowWindow;
       public short cbReserved2;
       public IntPtr lpReserved2;
       public IntPtr hStdInput;
       public IntPtr hStdOutput;
       public IntPtr hStdError;
    }
 
    /// <summary>
    /// Size of buffer used when retrieving window names.
    /// </summary>
    public const int MaxWindowNameLength = 100;
 
    //
    // winAPI constants.
    //
    private const short SW_HIDE = 0;
    private const short SW_NORMAL = 1;
    private const int STARTF_USESTDHANDLES = 0x00000100;
    private const int STARTF_USESHOWWINDOW = 0x00000001;
    private const int UOI_NAME = 2;
    private const int STARTF_USEPOSITION = 0x00000004;
    private const int NORMAL_PRIORITY_CLASS = 0x00000020;
    private const long DESKTOP_CREATEWINDOW = 0x0002L;
    private const long DESKTOP_ENUMERATE = 0x0040L;
    private const long DESKTOP_WRITEOBJECTS = 0x0080L;
    private const long DESKTOP_SWITCHDESKTOP = 0x0100L;
    private const long DESKTOP_CREATEMENU = 0x0004L;
    private const long DESKTOP_HOOKCONTROL = 0x0008L;
    private const long DESKTOP_READOBJECTS = 0x0001L;
    private const long DESKTOP_JOURNALRECORD = 0x0010L;
    private const long DESKTOP_JOURNALPLAYBACK = 0x0020L;
    private const long AccessRights = DESKTOP_JOURNALRECORD | DESKTOP_JOURNALPLAYBACK | DESKTOP_CREATEWINDOW | DESKTOP_ENUMERATE | DESKTOP_WRITEOBJECTS | DESKTOP_SWITCHDESKTOP | DESKTOP_CREATEMENU | DESKTOP_HOOKCONTROL | DESKTOP_READOBJECTS;
 
    /// <summary>
    /// Stores window handles and titles.
    /// </summary>
    public struct Window
    {
       private IntPtr m_handle;
       private string m_text;
 
       /// <summary>
       /// Gets the window handle.
       /// </summary>
       public IntPtr Handle
       {
          get
          {
             return m_handle;
          }
       }
 
       /// <summary>
       /// Gets teh window title.
       /// </summary>
       public string Text
       {
          get
          {
             return m_text;
          }
       }
 
       /// <summary>
       /// Creates a new window object.
       /// </summary>
       /// <param name="handle">Window handle.</param>
       /// <param name="text">Window title.</param>
       public Window(IntPtr handle, string text)
       {
          m_handle = handle;
          m_text = text;
       }
    }
 
    private IntPtr m_desktop;
    private string m_desktopName;
    private static List<String> m_sc;
    private List<IntPtr> m_windows;
    private bool m_disposed;
 
    /// <summary>
    /// Gets if a desktop is open.
    /// </summary>
    public bool IsOpen
    {
       get
       {
          return (m_desktop != IntPtr.Zero);
       }
    }
 
    /// <summary>
    /// Gets the name of the desktop, returns null if no desktop is open.
    /// </summary>
    public string DesktopName
    {
       get
       {
          return m_desktopName;
       }
    }
 
    /// <summary>
    /// Gets a handle to the desktop, IntPtr.Zero if no desktop open.
    /// </summary>
    public IntPtr DesktopHandle
    {
       get
       {
          return m_desktop;
       }
    }
 
    /// <summary>
    /// Opens the default desktop.
    /// </summary>
    public static readonly Desktop Default = Desktop.OpenDefault();
 
    /// <summary>
    /// Opens the desktop the user if viewing.
    /// </summary>
    public static readonly Desktop Input = Desktop.OpenInput();
 
    /// <summary>
    /// Creates a new Desktop object.
    /// </summary>
    public Desktop()
    {
       // init variables.
       m_desktop = IntPtr.Zero;
       m_desktopName = String.Empty;
       m_windows = new List<IntPtr>();
       m_disposed = false;
    }
    
    /// <summary>
    /// Creates a new Desktop object.
    /// </summary>
    public Desktop(string name)
    {
       // init variables.
       m_desktop = IntPtr.Zero;
       m_desktopName = name;
       m_windows = new List<IntPtr>();
       m_disposed = false;
    }
 
    // constructor is private to prevent invalid handles being passed to it.
    private Desktop(IntPtr desktop)
    {
       // init variables.
       m_desktop = desktop;
       m_desktopName = Desktop.GetName(desktop);
       m_windows = new List<IntPtr>();
       m_disposed = false;
    }
 
    ~Desktop()
    {
       // clean up, close the desktop.
       Close();
    }
 
 
    
    /// <summary>
    /// Closes the handle to a desktop.
    /// </summary>
    /// <returns>True if an open handle was successfully closed.</returns>
    public bool Close()
    {
       // make sure object isnt disposed.
       CheckDisposed();
 
       // check there is a desktop open.
       if (m_desktop != IntPtr.Zero)
       {
          // close the desktop.
          bool result = CloseDesktop(m_desktop);
 
          if (result)
          {
             m_desktop = IntPtr.Zero;
 
             m_desktopName = String.Empty;
          }
 
          return result;
       }
 
       // no desktop was open, so desktop is closed.
       return true;
    }
 
    /// <summary>
    /// Opens a desktop.
    /// </summary>
    /// <param name="name">The name of the desktop to open.</param>
    /// <returns>True if the desktop was successfully opened.</returns>
    public bool Open()
    {
       // make sure object isnt disposed.
       CheckDisposed();
 
       // close the open desktop.
       if (m_desktop != IntPtr.Zero)
       {
          // attempt to close the desktop.
          if (!Close()) return false;
       }
 
       // open the desktop.
       m_desktop = OpenDesktop(m_desktopName, 0, true, AccessRights);
 
       // something went wrong.
       if (m_desktop == IntPtr.Zero) return false;
 
       return true;
    }
 
    /// <summary>
    /// Switches input to the currently opened desktop.
    /// </summary>
    /// <returns>True if desktops were successfully switched.</returns>
    public bool Show()
    {
       // make sure object isnt disposed.
       CheckDisposed();
 
       // make sure there is a desktop to open.
       if (m_desktop == IntPtr.Zero) return false;
 
       // attempt to switch desktops.
       bool result = SwitchDesktop(m_desktop);
 
       return result;
    }
 
    /// <summary>
    /// Enumerates the windows on a desktop.
    /// </summary>
    /// <param name="windows">Array of Desktop.Window objects to recieve windows.</param>
    /// <returns>A window colleciton if successful, otherwise null.</returns>
    public List<Window> GetWindows()
    {
       // make sure object isnt disposed.
       CheckDisposed();
 
       // make sure a desktop is open.
       if (!IsOpen) return null;
 
       // init the List.
       m_windows.Clear();
       List<Window> windows = new List<Window>();
 
       // get windows.
       bool result = EnumDesktopWindows(m_desktop, new EnumDesktopWindowsProc(DesktopWindowsProc), IntPtr.Zero);
 
       // check for error.
       if (!result) return null;
 
       // get window names.
       IntPtr ptr = Marshal.AllocHGlobal(MaxWindowNameLength);
 
       foreach(IntPtr wnd in m_windows)
       {
          GetWindowText(wnd, ptr, MaxWindowNameLength);
          windows.Add(new Window(wnd, Marshal.PtrToStringAnsi(ptr)));
       }
 
       Marshal.FreeHGlobal(ptr);
 
       return windows;
    }
 
    private bool DesktopWindowsProc(IntPtr wndHandle, IntPtr lParam)
    {
       // add window handle to colleciton.
       m_windows.Add(wndHandle);
 
       return true;
    }
 
    /// <summary>
    /// Creates a new process in a desktop.
    /// </summary>
    /// <param name="path">Path to application.</param>
    /// <returns>The process object for the newly created process.</returns>
    public Process CreateProcess(string path)
    {
       return CreateProcess(path, null);
    }
    
    /// <summary>
    /// Creates a new process in a desktop.
    /// </summary>
    /// <param name="path">Path to application.</param>
    /// <param name="commandLineParameters">Arguments for the application.</param>
    /// <returns>The process object for the newly created process.</returns>
    public Process CreateProcess(string path, string commandLineParameters)
    {
       // make sure object isnt disposed.
       CheckDisposed();
       return Desktop.CreateProcess(path,commandLineParameters,m_desktopName);
    }
 
    /// <summary>
    /// Prepares a desktop for use by launching the Explorer Shell.  For use only on newly created desktops, call straight after CreateDesktop.
    /// </summary>
    public void Prepare()
    {
       // make sure object isnt disposed.
       CheckDisposed();
 
       // make sure a desktop is open.
       if (IsOpen)
       {
          // load explorer.
          CreateProcess("explorer.exe");
       }
    }
    
    /// <summary>
    /// Gets an array of all the processes running on this desktop.
    /// </summary>
    /// <returns>An array of the processes.</returns>
    public Process[] GetProcesses()
    {
       return GetProcesses( m_desktopName, StringComparison.InvariantCulture);
    }
 
    /// <summary>
    /// Enumerates all of the desktops.
    /// </summary>
    /// <param name="desktops">String array to recieve desktop names.</param>
    /// <returns>True if desktop names were successfully enumerated.</returns>
    public static string[] GetDesktops()
    {
       // attempt to enum desktops.
       IntPtr windowStation = GetProcessWindowStation();
 
       // check we got a valid handle.
       if (windowStation == IntPtr.Zero) return new string[0];
 
       string[] desktops;
 
       // lock the object. thread safety and all.
       lock(m_sc = new List<String>())
       {
          bool result = EnumDesktops(windowStation, new EnumDesktopProc(DesktopProc), IntPtr.Zero);
 
          // something went wrong.
          if (!result) return new string[0];
 
          //	// turn the collection into an array.
          desktops = new string[m_sc.Count];
          for(int i = 0; i < desktops.Length; i++) desktops[i] = m_sc[i];
       }
 
       return desktops;
    }
 
    private static bool DesktopProc(string lpszDesktop, IntPtr lParam)
    {
       // add the desktop to the collection.
       m_sc.Add(lpszDesktop);
 
       return true;
    }
 
    /// <summary>
    /// Switches to the specified desktop.
    /// </summary>
    /// <param name="name">Name of desktop to switch input to.</param>
    /// <returns>True if desktops were successfully switched.</returns>
    public static bool Show(string name)
    {
       // attmempt to open desktop.
       bool result = false;
 
       using (Desktop d = new Desktop(name))
       {
          result = d.Open();
 
          // something went wrong.
          if (!result) return false;
 
          // attempt to switch desktops.
          result = d.Show();
       }
 
       return result;
    }
 
    /// <summary>
    /// Gets the desktop of the calling thread.
    /// </summary>
    /// <returns>Returns a Desktop object for the valling thread.</returns>
    public static Desktop GetCurrent()
    {
       // get the desktop.
       return new Desktop(GetThreadDesktop(System.Threading.Thread.CurrentThread.ManagedThreadId));
    }
 
    /// <summary>
    /// Sets the desktop of the calling thread.
    /// NOTE: Function will fail if thread has hooks or windows in the current desktop.
    /// </summary>
    /// <param name="desktop">Desktop to put the thread in.</param>
    /// <returns>True if the threads desktop was successfully changed.</returns>
    public static bool SetCurrent(Desktop desktop)
    {
       // set threads desktop.
       if (!desktop.IsOpen) return false;
       return SetThreadDesktop(desktop.DesktopHandle);
    }
 
    /// <summary>
    /// Opens a desktop.
    /// </summary>
    /// <param name="name">The name of the desktop to open.</param>
    /// <returns>If successful, a Desktop object, otherwise, null.</returns>
    public static Desktop Open(string name)
    {
       // open the desktop.
       Desktop desktop = new Desktop(name);
       bool result = desktop.Open();
 
       // something went wrong.
       if (!result) return null;
 
       return desktop;
    }
 
    /// <summary>
    /// Opens the current input desktop.
    /// </summary>
    /// <returns>If successful, a Desktop object, otherwise, null.</returns>
    public static Desktop OpenInput()
    {
       // open the desktop.         
       IntPtr deskptr = OpenInputDesktop(0, true, AccessRights);
       if (deskptr == IntPtr.Zero) return null;
       
       return new Desktop(deskptr);
    }
 
    /// <summary>
    /// Opens the default desktop.
    /// </summary>
    /// <returns>If successful, a Desktop object, otherwise, null.</returns>
    public static Desktop OpenDefault()
    {
       // opens the default desktop.
       return Desktop.Open("Default");
    }
 
    /// <summary>
    /// Creates a new desktop.
    /// </summary>
    /// <param name="name">The name of the desktop to create.  Names are case sensitive.</param>
    /// <returns>If successful, a Desktop object, otherwise, null.</returns>
    public static Desktop Create(string name)
    {
       // make sure desktop doesnt already exist.
       if (Desktop.Exists(name))
       {
          // it exists, so open it.
          return Open(name);
       }
 
       // attempt to create desktop.
       IntPtr deskptr = CreateDesktop(name, IntPtr.Zero, IntPtr.Zero, 0, AccessRights, IntPtr.Zero);
       if (deskptr == IntPtr.Zero) return null;
 
       // open the desktop.
       return Open(name);
    }
 
    /// <summary>
    /// Gets the name of a given desktop.
    /// </summary>
    /// <param name="desktop">Desktop object whos name is to be found.</param>
    /// <returns>If successful, the desktop name, otherwise, null.</returns>
    public static string GetName(Desktop desktop)
    {
       // get name.
       if (desktop.IsOpen) return null;
 
       return GetName(desktop.DesktopHandle);
    }
 
    
    public static Win32Exception LastError;
    
    /// <summary>
    /// Gets the name of a desktop from a desktop handle.
    /// </summary>
    /// <param name="desktopHandle"></param>
    /// <returns>If successful, the desktop name, otherwise, null.</returns>
    public static string GetName(IntPtr desktopHandle)
    {
       // check its not a null pointer.
       // null pointers wont work.
       if (desktopHandle == IntPtr.Zero) return null;
 
       // get the length of the name.
       int needed = 0;
       string name = null;
       // always returns false, because we pass 0 for available size
       GetUserObjectInformation(desktopHandle, UOI_NAME, IntPtr.Zero, 0, ref needed);
 
       // get the name.
       IntPtr ptr = Marshal.AllocHGlobal(needed);
       if(!GetUserObjectInformation(desktopHandle, UOI_NAME, ptr, needed, ref needed)) {
          Marshal.FreeHGlobal(ptr);
          LastError = new Win32Exception();
       } else {
          name = Marshal.PtrToStringAnsi(ptr);
          Marshal.FreeHGlobal(ptr);
       }
       return name;
    }
 
    /// <summary>
    /// Checks if the specified desktop exists (using a case sensitive search).
    /// </summary>
    /// <param name="name">The name of the desktop.</param>
    /// <returns>True if the desktop exists, otherwise false.</returns>
    public static bool Exists(string name)
    {
       return Desktop.Exists(name, StringComparison.InvariantCultureIgnoreCase);
    }
 
    /// <summary>
    /// Checks if the specified desktop exists.
    /// </summary>
    /// <param name="name">The name of the desktop.</param>
    /// <param name="comparisonType">The type of string comparison to do.</param>
    /// <returns>True if the desktop exists, otherwise false.</returns>
    public static bool Exists(string name, StringComparison comparisonType)
    {
       // enumerate desktops.
       string[] desktops = Desktop.GetDesktops();
 
       // return true if desktop exists.
       foreach(string desktop in desktops)
       {
          if(desktop.Equals( name, comparisonType )) return true;
       }
 
       return false;
    }
 
    /// <summary>
    /// Creates a new process on the specified desktop.
    /// </summary>
    /// <param name="path">Path to application.</param>
    /// <param name="commandLineParameters">Arguments for the application.</param>
    /// <param name="desktop">Desktop name.</param>
    /// <returns>A Process object for the newly created process, otherwise, null.</returns>
    public static Process CreateProcess(string path, string commandLineParameters, string desktop)
    {
       if (!Desktop.Exists(desktop)) return null;
 
       // set startup parameters.
       STARTUPINFO si = new STARTUPINFO();
       si.cb = Marshal.SizeOf(si);
       si.lpDesktop = desktop;
 
       PROCESS_INFORMATION pi = new PROCESS_INFORMATION();
 
       //StringBuilder lpPath = new StringBuilder(path);
       StringBuilder lpCommandLine = new StringBuilder();
       lpCommandLine.Append("\"");
       lpCommandLine.Append(path);
       lpCommandLine.Append("\" ");
       lpCommandLine.Append(commandLineParameters);
       // lpPath.EnsureCapacity(256);
       // create the process.
       if(!CreateProcess(null, lpCommandLine.ToString(), IntPtr.Zero, IntPtr.Zero, true, NORMAL_PRIORITY_CLASS, IntPtr.Zero, null, ref si, out pi) ) {
          throw new Win32Exception();
       }
       
       // Get the process.
       return Process.GetProcessById(pi.dwProcessId);
       
    }
 
 
    
    /// <summary>
    /// Gets an array of all the processes running on the Input desktop.
    /// </summary>
    /// <returns>An array of the processes.</returns>
    public static Process[] GetInputProcesses()
    {
       return GetProcesses( GetName(Desktop.Input.DesktopHandle), StringComparison.InvariantCulture);
    }  
    
    /// <summary>
    /// Gets an array of all the processes running on the specified desktop.
    /// </summary>
    /// <param name="desktop">The name of the desktop for which to return processes</param>
    /// <returns>An array of the processes.</returns>
    public static Process[] GetProcesses(string desktop)
    {
       return GetProcesses( desktop, StringComparison.InvariantCultureIgnoreCase);
    }      
    
    /// <summary>
    /// Gets an array of all the processes running on the specified desktop (using the speficied string comparison)
    /// </summary>
    /// <param name="desktop">The name of the desktop for which to return processes</param>
    /// <param name="comparisonType">The type of string comparison to do</param>
    /// <returns>An array of the processes.</returns>
    public static Process[] GetProcesses(string desktop, StringComparison comparisonType)
    {
       // get all processes.
       Process[] processes = Process.GetProcesses();
 
       List<Process> procs = new List<Process>();
 
       // cycle through the processes.
       foreach(Process process in processes)
       {
          // check the threads of the process - are they in this one?
          foreach(ProcessThread pt in process.Threads)
          {
             string deskname = GetName(GetThreadDesktop(pt.Id));
             if(deskname == null) continue;
             
             // check for a desktop name match.
             if (deskname.Equals(desktop,comparisonType))
             {
                // found a match, add to list, and bail.
                procs.Add(process);
                break;
             } else {
                Console.WriteLine("Wow, Really? The '" + deskname + "' desktop showed up!");
             }
          }
       }
       return procs.ToArray();
    }
 
    /// <summary>
    /// Dispose Object.
    /// </summary>
    public void Dispose()
    {
       // dispose
       Dispose(true);
 
       // suppress finalisation
       GC.SuppressFinalize(this);
    }
 
    /// <summary>
    /// Dispose Object.
    /// </summary>
    /// <param name="disposing">True to dispose managed resources.</param>
    public virtual void Dispose(bool disposing)
    {
       if (!m_disposed)
       {
          // dispose of managed resources,
          // close handles
          Close();
       }
 
       m_disposed = true;
    }
 
    private void CheckDisposed()
    {
       // check if disposed
       if (m_disposed)
       {
          // object disposed, throw exception
          throw new ObjectDisposedException("");
       }
    }
 
    /// <summary>
    /// Creates a new Desktop object with the same desktop open.
    /// </summary>
    /// <returns>Cloned desktop object.</returns>
    public object Clone()
    {
       // make sure object isnt disposed.
       CheckDisposed();
 
       Desktop desktop = new Desktop(m_desktopName);
 
       // if a desktop is open, make the clone open it.
       if (IsOpen) desktop.Open();
 
       return desktop;
    }
 
    /// <summary>
    /// Gets the desktop name.
    /// </summary>
    /// <returns>The desktop name, or a blank string if no desktop open.</returns>
    public override string ToString()
    {
       // return the desktop name.
       return m_desktopName;
    }
 }
`

