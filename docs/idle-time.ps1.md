---
Author: greg zakharov
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 6251
Published Date: 2016-03-15t06
Archived Date: 2016-08-01t05
---

# idle time - 

## Description

see http

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 using System;
 using System.Drawing;
 using System.Reflection;
 using System.Windows.Forms;
 using System.Runtime.InteropServices;
 
 [assembly: AssemblyTitle("Idle Time")]
 [assembly: AssemblyVersion("2.0.0.0")]
 
 namespace IdleTime {
   internal sealed class AssemblyInfo {
     internal Type t;
     internal AssemblyInfo() { t = typeof(frmMain); }
     
     internal string Title {
       get {
         Object[] a = t.Assembly.GetCustomAttributes(typeof(AssemblyTitleAttribute), false);
         return ((AssemblyTitleAttribute)a[0]).Title;
       }
     }
   }
   
   internal static class WinAPI {
     [StructLayout(LayoutKind.Sequential)]
     internal struct PLASTINPUTINFO {
       public uint cbSize;
       public uint dwTime;
     }
     
     [DllImport("user32.dll")]
     [return: MarshalAs(UnmanagedType.Bool)]
     internal static extern bool GetLastInputInfo(ref PLASTINPUTINFO plii);
     
     internal static DateTime LastInput {
       get {
         PLASTINPUTINFO lii = new PLASTINPUTINFO();
         lii.cbSize = (uint)Marshal.SizeOf(typeof(PLASTINPUTINFO));
         GetLastInputInfo(ref lii);
         DateTime bTime = DateTime.Now.AddMilliseconds(-Environment.TickCount);
         DateTime lTime = bTime.AddMilliseconds(lii.dwTime);
         
         return lTime;
       }
     }
     
     internal static TimeSpan IdleTime {
       get {
         return DateTime.Now.Subtract(LastInput);
       }
     }
   }
   
   internal sealed class frmMain : Form {
     public frmMain() {
       InitializeComponent();
       this.Text = (new AssemblyInfo()).Title;
     }
 
     private Label lblLabel1;
     private Label lblLabel2;
     private Timer tmrUpdate;
     
     private void InitializeComponent() {
       this.lblLabel1 = new Label();
       this.lblLabel2 = new Label();
       this.tmrUpdate = new Timer();
       //
       //lblLabel1
       //
       this.lblLabel1.Location = new Point(13, 13);
       this.lblLabel1.Size = new Size(200, 19);
       this.lblLabel1.Text = "Idle Time: " + WinAPI.IdleTime.ToString();
       //
       //lblLabel2
       //
       this.lblLabel2.Location = new Point(13, 33);
       this.lblLabel2.Size = new Size(200, 19);
       this.lblLabel2.Text = "Last Input: " + WinAPI.LastInput.ToString();
       //
       //tmrUpdate
       //
       this.tmrUpdate.Enabled = true;
       this.tmrUpdate.Interval = 1000;
       this.tmrUpdate.Tick += new EventHandler(tmrUpdate_Tick);
       //
       //frmMain
       //
       this.ClientSize = new Size(300, 70);
       this.Controls.AddRange(new Control[] {this.lblLabel1, this.lblLabel2});
       this.FormBorderStyle = FormBorderStyle.FixedSingle;
       this.MaximizeBox = false;
       this.StartPosition = FormStartPosition.CenterScreen;
     }
     
     private void tmrUpdate_Tick(object sender, EventArgs e) {
       lblLabel1.Text = lblLabel2.Text = String.Empty;
       lblLabel1.Text = "Idle Time: " + WinAPI.IdleTime.ToString();
       lblLabel2.Text = "Last Input: " + WinAPI.LastInput.ToString();
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

