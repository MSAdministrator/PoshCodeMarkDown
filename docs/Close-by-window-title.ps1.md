---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 4088
Published Date: 2016-04-08t14
Archived Date: 2016-02-22t16
---

# close by window title - 

## Description

to close app

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 using System;
 using System.Text;
 using System.Drawing;
 using System.Reflection;
 using System.Diagnostics;
 using System.Windows.Forms;
 using System.ComponentModel;
 using System.Runtime.InteropServices;
 
 [assembly: AssemblyTitle("CloseByTitle")]
 [assembly: AssemblyVersion("3.5.0.0")]
 
 namespace CloseByTitle {
   internal static class WinAPI {
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool EnumWindows(EnumWindowsProc lpEnumFunc, IntPtr lParam);
 
     [DllImport("user32.dll", CharSet = CharSet.Unicode)]
     internal static extern int GetWindowText(IntPtr hWnd, StringBuilder lpString, int nMaxCount);
 
     [DllImport("user32.dll")]
     internal static extern int GetWindowTextLength(IntPtr hWnd);
 
     [DllImport("user32.dll")]
     internal static extern int GetWindowThreadProcessId(IntPtr hWnd, out int lpdwProcessId);
 
     internal delegate bool EnumWindowsProc(IntPtr hWnd, IntPtr lParam);
   }
 
   internal class frmMain : Form {
     public frmMain() {
       InitializeComponent();
     }
 
     private TextBox txtTitle;
     private Button  btnClose;
 
     private void InitializeComponent() {
       this.txtTitle = new TextBox();
       this.btnClose = new Button();
       //
       //txtTitle
       //
       this.txtTitle.Location = new Point(16, 16);
       this.txtTitle.Width = 190;
       //
       //btnClose
       //
       this.btnClose.Location = new Point(220, 15);
       this.btnClose.Text = "Close it!";
       this.btnClose.Click += new EventHandler(btnClose_Click);
       //
       //frmMain
       //
       this.Controls.AddRange(new Control[] {this.txtTitle, this.btnClose});
       this.ClientSize = new Size(310, 65);
       this.MaximizeBox = false;
       this.MinimizeBox = false;
       this.StartPosition = FormStartPosition.CenterScreen;
       this.Load += new EventHandler(frmMain_Load);
     }
 
     private static string GetText(IntPtr hWnd) {
       int len = WinAPI.GetWindowTextLength(hWnd);
       StringBuilder sb = new StringBuilder(len + 1);
       len = WinAPI.GetWindowText(hWnd, sb, sb.Capacity);
 
       return sb.ToString(0, len);
     }
 
     private void btnClose_Click(object sender, EventArgs e) {
       int pid, res;
       string search = txtTitle.Text, window = null;
 
       if (!WinAPI.EnumWindows(new WinAPI.EnumWindowsProc((hWnd, lParam) => {
         window = GetText(hWnd);
 
         if (!string.IsNullOrEmpty(window) && window.Contains(search)) {
           res = WinAPI.GetWindowThreadProcessId(hWnd, out pid);
           //there is an exception if rights are not enough
           if (res != 0) Process.GetProcessById(pid).Kill();
         }
 
         return true;
       }), IntPtr.Zero)) {
         throw new Win32Exception(Marshal.GetLastWin32Error());
       }
     }
 
     private void frmMain_Load(object sender, EventArgs e) {
       Assembly assem = typeof(frmMain).Assembly;
       string title = ((AssemblyTitleAttribute)Attribute.GetCustomAttribute(assem,
                                           typeof(AssemblyTitleAttribute))).Title;
       this.Text = title;
     }
   }
 
   internal sealed class Program {
     [STAThread]
     static void Main() {
       Application.EnableVisualStyles();
       Application.Run(new frmMain());
     }
   }
 }
`

