---
Author: bryan jaudon
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 3733
Published Date: 2012-10-31t13
Archived Date: 2012-11-04t00
---

# posh email tester gui - 

## Description

gui front-end for send-mailmessage set up as a testing program. useful for email administrators who want to send a quick test email.

## Comments



## Usage



## TODO



## script

`openfile-dialog`

## Code

`#
 #
 <#
 
     .NOTES
     Name     : PoSh Email Tester GUI
     Author   : Bryan Jaudon <bryan.jaudon@gmail.com>
     Version  : 1.0
     Date     : 10/30/2012
 
     .Description
     GUI Email tester program utilizing PowerShell Send-MailMessage cmdlet.
 
 #>
 
 
 $code = @"
 using System;
 using System.Drawing;
 using System.Runtime.InteropServices;
 
 namespace System
 {
 	public class IconExtractor
 	{
 
 	 public static Icon Extract(string file, int number, bool largeIcon)
 	 {
 	  IntPtr large;
 	  IntPtr small;
 	  ExtractIconEx(file, number, out large, out small, 1);
 	  try
 	  {
 	   return Icon.FromHandle(largeIcon ? large : small);
 	  }
 	  catch
 	  {
 	   return null;
 	  }
 
 	 }
 	 [DllImport("Shell32.dll", EntryPoint = "ExtractIconExW", CharSet = CharSet.Unicode, ExactSpelling = true, CallingConvention = CallingConvention.StdCall)]
 	 private static extern int ExtractIconEx(string sFile, int iIndex, out IntPtr piLargeVersion, out IntPtr piSmallVersion, int amountIcons);
 
 	}
 }
 "@
 
 Add-Type -TypeDefinition $code -ReferencedAssemblies System.Drawing
 
 [reflection.assembly]::loadwithpartialname("System.Windows.Forms") | Out-Null
 [reflection.assembly]::loadwithpartialname("System.Drawing") | Out-Null
 [System.Windows.Forms.Application]::EnableVisualStyles()
 
 function GenerateForm {
 ########################################################################
 ########################################################################
 
 $frmTester = New-Object System.Windows.Forms.Form
 $lnkCredential = New-Object System.Windows.Forms.LinkLabel
 $lblCredential = New-Object System.Windows.Forms.Label
 $cbSSL = New-Object System.Windows.Forms.CheckBox
 $tbSMTPServer = New-Object System.Windows.Forms.TextBox
 $lblSMTPServer = New-Object System.Windows.Forms.Label
 $lblAttachment = New-Object System.Windows.Forms.Label
 $btnBrowse = New-Object System.Windows.Forms.Button
 $tbAttachment = New-Object System.Windows.Forms.TextBox
 $btnSend = New-Object System.Windows.Forms.Button
 $btnExit = New-Object System.Windows.Forms.Button
 $tbBody = New-Object System.Windows.Forms.TextBox
 $tbSubject = New-Object System.Windows.Forms.TextBox
 $lblSubject = New-Object System.Windows.Forms.Label
 $tbFrom = New-Object System.Windows.Forms.TextBox
 $lblFrom = New-Object System.Windows.Forms.Label
 $tbTo = New-Object System.Windows.Forms.TextBox
 $lblTo = New-Object System.Windows.Forms.Label
 $pbIcon = New-Object System.Windows.Forms.PictureBox
 $lblTitle = New-Object System.Windows.Forms.Label
 $openFileDialog1 = New-Object System.Windows.Forms.OpenFileDialog
 $InitialFormWindowState = New-Object System.Windows.Forms.FormWindowState
 
 #----------------------------------------------
 #----------------------------------------------
 
 
 $handler_lnkCredential_LinkClicked= 
 {
     try { $script:Credentials=Get-Credential -Message "Please enter your SMTP Server credentials." }
     catch { $script:Credentials=$null }
     finally { 
         if ($script:Credentials -eq $null) { $lnkCredential.Text = "Anonymous" }
         else { $lnkCredential.Text = $script:Credentials.UserName }
     }
 
 }
 
 $btnBrowse_OnClick= 
 {
     $tbAttachment.Text=OpenFile-Dialog
 }
 
 $btnSend_OnClick= 
 {
 
     $GUID=[guid]::NewGuid()
 
     try { SendMail -To $tbTo.Text -From $tbFrom.Text -Subject $tbSubject.Text -SMTPServer $tbSMTPServer.Text -Attachment $tbAttachment.Text -Body "$($tbBody.Text)`r`n`r`nEmail ID: $GUID" -SSL $cbSSL.Checked -Credential $Script:Credentials }
     catch { ShowPrompt -Message $_ -Icon Error -Buttons OK; return }
     ShowPrompt -Message "Email was sent successfully!`n`nEmail ID: $GUID" -Icon Information -Buttons OK
 }
 
 $OnLoadForm_StateCorrection=
 	$frmTester.WindowState = $InitialFormWindowState
 }
 
 #----------------------------------------------
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 469
 $System_Drawing_Size.Width = 553
 $frmTester.ClientSize = $System_Drawing_Size
 $frmTester.DataBindings.DefaultDataSourceUpdateMode = 0
 $frmTester.FormBorderStyle = 1
 $frmTester.Name = "frmTester"
 $frmTester.Text = "PoSh Email Tester"
 $frmTester.Icon = [System.IconExtractor]::Extract("shell32.dll",156,$true)
 
 $lnkCredential.AutoEllipsis = $True
 $lnkCredential.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 392
 $System_Drawing_Point.Y = 68
 $lnkCredential.Location = $System_Drawing_Point
 $lnkCredential.Name = "lnkCredential"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 146
 $lnkCredential.Size = $System_Drawing_Size
 $lnkCredential.TabIndex = 1
 $lnkCredential.TabStop = $True
 $lnkCredential.Text = "Anonymous"
 $lnkCredential.TextAlign = 16
 $lnkCredential.VisitedLinkColor = [System.Drawing.Color]::FromArgb(255,0,0,255)
 $lnkCredential.add_LinkClicked($handler_lnkCredential_LinkClicked)
 
 $frmTester.Controls.Add($lnkCredential)
 
 $lblCredential.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 327
 $System_Drawing_Point.Y = 68
 $lblCredential.Location = $System_Drawing_Point
 $lblCredential.Name = "lblCredential"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 59
 $lblCredential.Size = $System_Drawing_Size
 $lblCredential.TabIndex = 15
 $lblCredential.Text = "Credential:"
 $lblCredential.TextAlign = 16
 
 $frmTester.Controls.Add($lblCredential)
 
 
 $cbSSL.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 434
 $cbSSL.Location = $System_Drawing_Point
 $cbSSL.Name = "cbSSL"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 140
 $cbSSL.Size = $System_Drawing_Size
 $cbSSL.TabIndex = 6
 $cbSSL.Text = "Send &using SSL/TLS"
 $cbSSL.UseVisualStyleBackColor = $True
 
 $frmTester.Controls.Add($cbSSL)
 
 $tbSMTPServer.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 94
 $System_Drawing_Point.Y = 71
 $tbSMTPServer.Location = $System_Drawing_Point
 $tbSMTPServer.Name = "tbSMTPServer"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 216
 $tbSMTPServer.Size = $System_Drawing_Size
 $tbSMTPServer.TabIndex = 0
 
 $frmTester.Controls.Add($tbSMTPServer)
 
 $lblSMTPServer.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 69
 $lblSMTPServer.Location = $System_Drawing_Point
 $lblSMTPServer.Name = "lblSMTPServer"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $lblSMTPServer.Size = $System_Drawing_Size
 $lblSMTPServer.TabIndex = 14
 $lblSMTPServer.Text = "SMTP Server:"
 $lblSMTPServer.TextAlign = 16
 
 $frmTester.Controls.Add($lblSMTPServer)
 
 $lblAttachment.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 204
 $lblAttachment.Location = $System_Drawing_Point
 $lblAttachment.Name = "lblAttachment"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $lblAttachment.Size = $System_Drawing_Size
 $lblAttachment.TabIndex = 13
 $lblAttachment.Text = "Attachment:"
 $lblAttachment.TextAlign = 16
 
 $frmTester.Controls.Add($lblAttachment)
 
 
 $btnBrowse.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 463
 $System_Drawing_Point.Y = 207
 $btnBrowse.Location = $System_Drawing_Point
 $btnBrowse.Name = "btnBrowse"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $btnBrowse.Size = $System_Drawing_Size
 $btnBrowse.TabIndex = 4
 $btnBrowse.Text = "B&rowse..."
 $btnBrowse.UseVisualStyleBackColor = $True
 $btnBrowse.add_Click($btnBrowse_OnClick)
 
 $frmTester.Controls.Add($btnBrowse)
 
 $tbAttachment.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 94
 $System_Drawing_Point.Y = 208
 $tbAttachment.Location = $System_Drawing_Point
 $tbAttachment.Name = "tbAttachment"
 $tbAttachment.ReadOnly = $True
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 363
 $tbAttachment.Size = $System_Drawing_Size
 $tbAttachment.TabIndex = 0
 $tbAttachment.TabStop = $False
 
 $frmTester.Controls.Add($tbAttachment)
 
 
 $btnSend.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 382
 $System_Drawing_Point.Y = 434
 $btnSend.Location = $System_Drawing_Point
 $btnSend.Name = "btnSend"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $btnSend.Size = $System_Drawing_Size
 $btnSend.TabIndex = 7
 $btnSend.Text = "&Send"
 $btnSend.UseVisualStyleBackColor = $True
 $btnSend.add_Click($btnSend_OnClick)
 
 $frmTester.Controls.Add($btnSend)
 
 
 $btnExit.DataBindings.DefaultDataSourceUpdateMode = 0
 $btnExit.DialogResult = 2
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 463
 $System_Drawing_Point.Y = 434
 $btnExit.Location = $System_Drawing_Point
 $btnExit.Name = "btnExit"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $btnExit.Size = $System_Drawing_Size
 $btnExit.TabIndex = 8
 $btnExit.Text = "E&xit"
 $btnExit.UseVisualStyleBackColor = $True
 
 $frmTester.Controls.Add($btnExit)
 
 $tbBody.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 241
 $tbBody.Location = $System_Drawing_Point
 $tbBody.Multiline = $True
 $tbBody.Name = "tbBody"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 173
 $System_Drawing_Size.Width = 526
 $tbBody.Size = $System_Drawing_Size
 $tbBody.TabIndex = 5
 $tbBody.Text = "This is a test email message from PoSh Email Tester. "
 
 $frmTester.Controls.Add($tbBody)
 
 $tbSubject.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 94
 $System_Drawing_Point.Y = 167
 $tbSubject.Location = $System_Drawing_Point
 $tbSubject.Name = "tbSubject"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 445
 $tbSubject.Size = $System_Drawing_Size
 $tbSubject.TabIndex = 3
 $tbSubject.Text = "PoSh Email Tester - Test Email"
 
 $frmTester.Controls.Add($tbSubject)
 
 $lblSubject.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 165
 $lblSubject.Location = $System_Drawing_Point
 $lblSubject.Name = "lblSubject"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $lblSubject.Size = $System_Drawing_Size
 $lblSubject.TabIndex = 6
 $lblSubject.Text = "Subject:"
 $lblSubject.TextAlign = 16
 
 $frmTester.Controls.Add($lblSubject)
 
 $tbFrom.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 94
 $System_Drawing_Point.Y = 102
 $tbFrom.Location = $System_Drawing_Point
 $tbFrom.Name = "tbFrom"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 445
 $tbFrom.Size = $System_Drawing_Size
 $tbFrom.TabIndex = 1
 $tbFrom.Text = "test@test.com"
 
 $frmTester.Controls.Add($tbFrom)
 
 $lblFrom.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 100
 $lblFrom.Location = $System_Drawing_Point
 $lblFrom.Name = "lblFrom"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $lblFrom.Size = $System_Drawing_Size
 $lblFrom.TabIndex = 4
 $lblFrom.Text = "From:"
 $lblFrom.TextAlign = 16
 
 $frmTester.Controls.Add($lblFrom)
 
 $tbTo.DataBindings.DefaultDataSourceUpdateMode = 0
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 94
 $System_Drawing_Point.Y = 134
 $tbTo.Location = $System_Drawing_Point
 $tbTo.Name = "tbTo"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 20
 $System_Drawing_Size.Width = 445
 $tbTo.Size = $System_Drawing_Size
 $tbTo.TabIndex = 2
 
 $frmTester.Controls.Add($tbTo)
 
 $lblTo.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 132
 $lblTo.Location = $System_Drawing_Point
 $lblTo.Name = "lblTo"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 75
 $lblTo.Size = $System_Drawing_Size
 $lblTo.TabIndex = 2
 $lblTo.Text = "To:"
 $lblTo.TextAlign = 16
 
 $frmTester.Controls.Add($lblTo)
 
 
 $pbIcon.BackgroundImageLayout = 3
 $pbIcon.DataBindings.DefaultDataSourceUpdateMode = 0
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 13
 $System_Drawing_Point.Y = 9
 $pbIcon.Location = $System_Drawing_Point
 $pbIcon.Name = "pbIcon"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 48
 $System_Drawing_Size.Width = 48
 $pbIcon.Size = $System_Drawing_Size
 $pbIcon.SizeMode = 1
 $pbIcon.TabIndex = 1
 $pbIcon.TabStop = $False
 $pbIcon.Image = [System.IconExtractor]::Extract("shell32.dll",156,$true)
 
 $frmTester.Controls.Add($pbIcon)
 
 $lblTitle.DataBindings.DefaultDataSourceUpdateMode = 0
 $lblTitle.Font = New-Object System.Drawing.Font("Trebuchet MS",9.75,1,3,1)
 
 $System_Drawing_Point = New-Object System.Drawing.Point
 $System_Drawing_Point.X = 76
 $System_Drawing_Point.Y = 21
 $lblTitle.Location = $System_Drawing_Point
 $lblTitle.Name = "lblTitle"
 $System_Drawing_Size = New-Object System.Drawing.Size
 $System_Drawing_Size.Height = 23
 $System_Drawing_Size.Width = 463
 $lblTitle.Size = $System_Drawing_Size
 $lblTitle.TabIndex = 0
 $lblTitle.Text = "PoSh Email Tester 1.0"
 $lblTitle.TextAlign = 16
 
 $frmTester.Controls.Add($lblTitle)
 
 $openFileDialog1.Filter = "All Files (*.*)|*.*"
 $openFileDialog1.ShowHelp = $True
 
 
 $InitialFormWindowState = $frmTester.WindowState
 $frmTester.add_Load($OnLoadForm_StateCorrection)
 $frmTester.ShowDialog()| Out-Null
 
 
 
 Function OpenFile-Dialog {
     $open=$OpenFileDialog1.ShowDialog()
     if ($open -eq "OK") { 
         
         return $OpenFileDialog1.FileName
         
     }
     else { return }
 }
 
 Function ShowPrompt {
     param ($Message,$Buttons,$Icon,$DefaultButton="button1")
     return [System.Windows.Forms.MessageBox]::Show($Message,$frmTester.Text,$Buttons,$Icon,$DefaultButton)
 }
 
 Function SendMail {
     param ($To,$From,$Subject,$Attachment,$SMTPServer,$Body,$SSL,$Credential)
     $MailMessageParams=@{
         "To"=$To
         "From"=$From
         "Subject"=$Subject
         "SMTPServer"=$SMTPServer
         "Body"=$Body
         "UseSSL"=$SSL
     }
     if ($Attachment) { $MailMessageParams.Add("Attachment",$Attachment) }
     if ($Credential) { $MailMessageParams.Add("Credential",$Credential) }
     Send-MailMessage @MailMessageParams -ErrorAction STOP
     
 }
 
 
 GenerateForm
`

