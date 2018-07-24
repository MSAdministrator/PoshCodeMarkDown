---
Author: kirk munro                                                                                        #
Publisher: quest software, inc.                                                                              #
Copyright: â© 2010 quest software, inc.. all rights reserved.                                                 #
Email: 
Version: 2.1.0.1200
Encoding: utf-8
License: cc0
PoshCode ID: 4843
Published Date: 2014-01-27t06
Archived Date: 2014-04-10t15
---

# add-on.publishonline.las - 

## Description

this file was uploaded by a powergui script editor add-on.

## Comments



## Usage

to load this module in your script editor

## TODO



## module

`publish-poshcodedocument`

## Code

`#
 #
 #######################################################################################################################
 #######################################################################################################################
 
 Set-StrictMode -Version 2
 
 
 if ($Host.Name â��ne 'PowerGUIScriptEditorHost') { return }
 if ($Host.Version -lt '2.1.0.1200') {
 	[System.Windows.Forms.MessageBox]::Show("The ""$(Split-Path -Path $PSScriptRoot -Leaf)"" Add-on module requires version 2.1.0.1200 or later of the Script Editor. The current Script Editor version is $($Host.Version).$([System.Environment]::NewLine * 2)Please upgrade to version 2.1.0.1200 and try again.","Version 2.1.0.1200 or later is required",[System.Windows.Forms.MessageBoxButtons]::OK,[System.Windows.Forms.MessageBoxIcon]::Information) | Out-Null
 	return
 }
 
 $se = [Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance
 
 
 
 $iconLibrary = @{
 	PublishOnlineIcon16 = New-Object System.Drawing.Icon -ArgumentList "$PSScriptRoot\Resources\PublishOnline.ico",16,16
 	PublishOnlineIcon32 = New-Object System.Drawing.Icon -ArgumentList "$PSScriptRoot\Resources\PublishOnline.ico",32,32
 }
 
 $imageLibrary = @{
 	PublishOnlineImage16 = $iconLibrary['PublishOnlineIcon16'].ToBitmap()
 	PublishOnlineImage32 = $iconLibrary['PublishOnlineIcon32'].ToBitmap()
 }
 
 
 
 $publishOnlineDialogCode = @'
 using System;
 using System.Windows.Forms;
 
 namespace Addon
 {
 	namespace PublishOnline
 	{
 		public partial class PublishOnlineForm : Form
 		{
 			public PublishOnlineForm()
 			{
 				InitializeComponent();
 				this.comboBoxDuration.SelectedIndex = 2;
 			}
 
 			private void buttonOK_Click(object sender, EventArgs e)
 			{
 				this.DialogResult = DialogResult.None;
 				if (this.textBoxTitle.Text.Trim().Length == 0)
 				{
 					MessageBox.Show(this, "You must provide a title for the document you are publishing.", "Title is required", MessageBoxButtons.OK, MessageBoxIcon.Error);
 					this.textBoxTitle.SelectAll();
 					this.textBoxTitle.Focus();
 				}
 				else if (this.textBoxAuthor.Text.Trim().Length == 0)
 				{
 					MessageBox.Show(this, "You must provide an author for the document you are publishing.", "Author is required", MessageBoxButtons.OK, MessageBoxIcon.Error);
 					this.textBoxAuthor.SelectAll();
 					this.textBoxAuthor.Focus();
 				}
 				else if (this.textBoxTitle.Text.Trim().Length == 0)
 				{
 					MessageBox.Show(this, "You must provide a description for the document you are publishing.", "Description is required", MessageBoxButtons.OK, MessageBoxIcon.Error);
 					this.textBoxDescription.SelectAll();
 					this.textBoxDescription.Focus();
 				}
 				else
 				{
 					this.DialogResult = DialogResult.OK;
 				}
 			}
 
 			public System.Drawing.Image Image
 			{
 				set
 				{
 					this.pictureBoxIcon.Image = value;
 				}
 			}
 
 			public System.String Title
 			{
 				get
 				{
 					return this.textBoxTitle.Text.Trim();
 				}
 				set
 				{
 					if (value.Trim().Length > 0)
 					{
 						this.textBoxTitle.Text = value.Trim();
 					}
 				}
 			}
 
 			public System.String Author
 			{
 				get
 				{
 					return this.textBoxAuthor.Text.Trim();
 				}
 				set
 				{
 					if (value.Trim().Length > 0)
 					{
 						this.textBoxAuthor.Text = value.Trim();
 					}
 				}
 			}
 
 			public System.String Description
 			{
 				get
 				{
 					return this.textBoxDescription.Text.Trim();
 				}
 				set
 				{
 					if (value.Trim().Length > 0)
 					{
 						this.textBoxDescription.Text = value.Trim();
 					}
 				}
 			}
 
 			public System.String Duration
 			{
 				get
 				{
 					string selectedDuration = "Forever";
 					switch (this.comboBoxDuration.SelectedIndex)
 					{
 						case 0:
 							selectedDuration = "Day";
 							break;
 						case 1:
 							selectedDuration = "Month";
 							break;
 						default:
 							break;
 					}
 					return selectedDuration;
 				}
 				set
 				{
 					if (value.Trim().Length > 0)
 					{
 						switch (value.Trim())
 						{
 							case "Day":
 								this.comboBoxDuration.SelectedIndex = 0;
 								break;
 							case "Month":
 								this.comboBoxDuration.SelectedIndex = 1;
 								break;
 							default:
 								this.comboBoxDuration.SelectedIndex = 2;
 								break;
 						}
 					}
 				}
 			}
 
 			public System.Boolean ViewPublishedDocument
 			{
 				get
 				{
 					return this.checkBoxViewPublishedDocument.Checked;
 				}
 				set
 				{
 					this.checkBoxViewPublishedDocument.Checked = value;
 				}
 			}
 		}
 
 		partial class PublishOnlineForm
 		{
 			/// <summary>
 			/// Required designer variable.
 			/// </summary>
 			private System.ComponentModel.IContainer components = null;
 
 			/// <summary>
 			/// Clean up any resources being used.
 			/// </summary>
 			/// <param name="disposing">true if managed resources should be disposed; otherwise, false.</param>
 			protected override void Dispose(bool disposing)
 			{
 				if (disposing && (components != null))
 				{
 					components.Dispose();
 				}
 				base.Dispose(disposing);
 			}
 
 
 			/// <summary>
 			/// Required method for Designer support - do not modify
 			/// the contents of this method with the code editor.
 			/// </summary>
 			private void InitializeComponent()
 			{
 				System.ComponentModel.ComponentResourceManager resources = new System.ComponentModel.ComponentResourceManager(typeof(PublishOnlineForm));
 				this.textBoxTitle = new System.Windows.Forms.TextBox();
 				this.labelInstructions = new System.Windows.Forms.Label();
 				this.labelTitle = new System.Windows.Forms.Label();
 				this.labelDescription = new System.Windows.Forms.Label();
 				this.textBoxDescription = new System.Windows.Forms.TextBox();
 				this.labelAuthor = new System.Windows.Forms.Label();
 				this.textBoxAuthor = new System.Windows.Forms.TextBox();
 				this.labelDuration = new System.Windows.Forms.Label();
 				this.pictureBoxIcon = new System.Windows.Forms.PictureBox();
 				this.buttonOK = new System.Windows.Forms.Button();
 				this.buttonCancel = new System.Windows.Forms.Button();
 				this.comboBoxDuration = new System.Windows.Forms.ComboBox();
 				this.checkBoxViewPublishedDocument = new System.Windows.Forms.CheckBox();
 				((System.ComponentModel.ISupportInitialize)(this.pictureBoxIcon)).BeginInit();
 				this.SuspendLayout();
 				// 
 				// textBoxTitle
 				// 
 				this.textBoxTitle.Anchor = ((System.Windows.Forms.AnchorStyles)(((System.Windows.Forms.AnchorStyles.Top | System.Windows.Forms.AnchorStyles.Left)
 							| System.Windows.Forms.AnchorStyles.Right)));
 				this.textBoxTitle.Location = new System.Drawing.Point(107, 79);
 				this.textBoxTitle.Name = "textBoxTitle";
 				this.textBoxTitle.Size = new System.Drawing.Size(380, 20);
 				this.textBoxTitle.TabIndex = 2;
 				// 
 				// labelInstructions
 				// 
 				this.labelInstructions.Anchor = ((System.Windows.Forms.AnchorStyles)(((System.Windows.Forms.AnchorStyles.Top | System.Windows.Forms.AnchorStyles.Left)
 							| System.Windows.Forms.AnchorStyles.Right)));
 				this.labelInstructions.Location = new System.Drawing.Point(73, 19);
 				this.labelInstructions.Name = "labelInstructions";
 				this.labelInstructions.Size = new System.Drawing.Size(414, 40);
 				this.labelInstructions.TabIndex = 0;
 				this.labelInstructions.Text = "Please provide a title, author, description, and duration for the document that you are publishing online.  You may also indicate whether or not you want to view the document online once it has been published.";
 				// 
 				// labelTitle
 				// 
 				this.labelTitle.AutoSize = true;
 				this.labelTitle.Location = new System.Drawing.Point(12, 82);
 				this.labelTitle.Name = "labelTitle";
 				this.labelTitle.Size = new System.Drawing.Size(30, 13);
 				this.labelTitle.TabIndex = 1;
 				this.labelTitle.Text = "&Title:";
 				// 
 				// labelDescription
 				// 
 				this.labelDescription.AutoSize = true;
 				this.labelDescription.Location = new System.Drawing.Point(12, 150);
 				this.labelDescription.Name = "labelDescription";
 				this.labelDescription.Size = new System.Drawing.Size(63, 13);
 				this.labelDescription.TabIndex = 5;
 				this.labelDescription.Text = "&Description:";
 				// 
 				// textBoxDescription
 				// 
 				this.textBoxDescription.AcceptsReturn = true;
 				this.textBoxDescription.Anchor = ((System.Windows.Forms.AnchorStyles)(((System.Windows.Forms.AnchorStyles.Top | System.Windows.Forms.AnchorStyles.Left)
 							| System.Windows.Forms.AnchorStyles.Right)));
 				this.textBoxDescription.Location = new System.Drawing.Point(107, 147);
 				this.textBoxDescription.Multiline = true;
 				this.textBoxDescription.Name = "textBoxDescription";
 				this.textBoxDescription.Size = new System.Drawing.Size(380, 100);
 				this.textBoxDescription.TabIndex = 6;
 				// 
 				// labelAuthor
 				// 
 				this.labelAuthor.AutoSize = true;
 				this.labelAuthor.Location = new System.Drawing.Point(12, 116);
 				this.labelAuthor.Name = "labelAuthor";
 				this.labelAuthor.Size = new System.Drawing.Size(41, 13);
 				this.labelAuthor.TabIndex = 3;
 				this.labelAuthor.Text = "&Author:";
 				// 
 				// textBoxAuthor
 				// 
 				this.textBoxAuthor.Anchor = ((System.Windows.Forms.AnchorStyles)(((System.Windows.Forms.AnchorStyles.Top | System.Windows.Forms.AnchorStyles.Left)
 							| System.Windows.Forms.AnchorStyles.Right)));
 				this.textBoxAuthor.Location = new System.Drawing.Point(107, 113);
 				this.textBoxAuthor.Name = "textBoxAuthor";
 				this.textBoxAuthor.Size = new System.Drawing.Size(380, 20);
 				this.textBoxAuthor.TabIndex = 4;
 				// 
 				// labelDuration
 				// 
 				this.labelDuration.AutoSize = true;
 				this.labelDuration.Location = new System.Drawing.Point(12, 265);
 				this.labelDuration.Name = "labelDuration";
 				this.labelDuration.Size = new System.Drawing.Size(50, 13);
 				this.labelDuration.TabIndex = 7;
 				this.labelDuration.Text = "Du&ration:";
 				// 
 				// pictureBoxIcon
 				// 
 				this.pictureBoxIcon.Location = new System.Drawing.Point(23, 23);
 				this.pictureBoxIcon.Name = "pictureBoxIcon";
 				this.pictureBoxIcon.Size = new System.Drawing.Size(32, 32);
 				this.pictureBoxIcon.SizeMode = System.Windows.Forms.PictureBoxSizeMode.AutoSize;
 				this.pictureBoxIcon.TabIndex = 11;
 				this.pictureBoxIcon.TabStop = false;
 				// 
 				// buttonOK
 				// 
 				this.buttonOK.Anchor = ((System.Windows.Forms.AnchorStyles)((System.Windows.Forms.AnchorStyles.Bottom | System.Windows.Forms.AnchorStyles.Right)));
 				this.buttonOK.DialogResult = System.Windows.Forms.DialogResult.OK;
 				this.buttonOK.Location = new System.Drawing.Point(331, 337);
 				this.buttonOK.Name = "buttonOK";
 				this.buttonOK.Size = new System.Drawing.Size(75, 23);
 				this.buttonOK.TabIndex = 10;
 				this.buttonOK.Text = "OK";
 				this.buttonOK.UseVisualStyleBackColor = true;
 				this.buttonOK.Click += new System.EventHandler(this.buttonOK_Click);
 				// 
 				// buttonCancel
 				// 
 				this.buttonCancel.Anchor = ((System.Windows.Forms.AnchorStyles)((System.Windows.Forms.AnchorStyles.Bottom | System.Windows.Forms.AnchorStyles.Right)));
 				this.buttonCancel.DialogResult = System.Windows.Forms.DialogResult.Cancel;
 				this.buttonCancel.Location = new System.Drawing.Point(412, 337);
 				this.buttonCancel.Name = "buttonCancel";
 				this.buttonCancel.Size = new System.Drawing.Size(75, 23);
 				this.buttonCancel.TabIndex = 11;
 				this.buttonCancel.Text = "Cancel";
 				this.buttonCancel.UseVisualStyleBackColor = true;
 				// 
 				// comboBoxDuration
 				// 
 				this.comboBoxDuration.Anchor = ((System.Windows.Forms.AnchorStyles)(((System.Windows.Forms.AnchorStyles.Top | System.Windows.Forms.AnchorStyles.Left)
 							| System.Windows.Forms.AnchorStyles.Right)));
 				this.comboBoxDuration.DropDownStyle = System.Windows.Forms.ComboBoxStyle.DropDownList;
 				this.comboBoxDuration.FormattingEnabled = true;
 				this.comboBoxDuration.Items.AddRange(new object[] {
             "Automatically remove the document after one day",
             "Automatically remove the document after one month",
             "Publish as a permanent document"});
 				this.comboBoxDuration.Location = new System.Drawing.Point(107, 262);
 				this.comboBoxDuration.Name = "comboBoxDuration";
 				this.comboBoxDuration.Size = new System.Drawing.Size(380, 21);
 				this.comboBoxDuration.TabIndex = 8;
 				// 
 				// checkBoxViewPublishedDocument
 				// 
 				this.checkBoxViewPublishedDocument.AutoSize = true;
 				this.checkBoxViewPublishedDocument.Location = new System.Drawing.Point(107, 299);
 				this.checkBoxViewPublishedDocument.Name = "checkBoxViewPublishedDocument";
 				this.checkBoxViewPublishedDocument.Size = new System.Drawing.Size(241, 17);
 				this.checkBoxViewPublishedDocument.TabIndex = 9;
 				this.checkBoxViewPublishedDocument.Text = "&View the document once it is published online";
 				this.checkBoxViewPublishedDocument.UseVisualStyleBackColor = true;
 				// 
 				// PublishOnlineForm
 				// 
 				this.AcceptButton = this.buttonOK;
 				this.AutoScaleDimensions = new System.Drawing.SizeF(6F, 13F);
 				this.AutoScaleMode = System.Windows.Forms.AutoScaleMode.Font;
 				this.CancelButton = this.buttonCancel;
 				this.ClientSize = new System.Drawing.Size(499, 372);
 				this.ControlBox = false;
 				this.Controls.Add(this.checkBoxViewPublishedDocument);
 				this.Controls.Add(this.comboBoxDuration);
 				this.Controls.Add(this.buttonCancel);
 				this.Controls.Add(this.buttonOK);
 				this.Controls.Add(this.pictureBoxIcon);
 				this.Controls.Add(this.labelDuration);
 				this.Controls.Add(this.labelAuthor);
 				this.Controls.Add(this.textBoxAuthor);
 				this.Controls.Add(this.labelDescription);
 				this.Controls.Add(this.textBoxDescription);
 				this.Controls.Add(this.labelTitle);
 				this.Controls.Add(this.labelInstructions);
 				this.Controls.Add(this.textBoxTitle);
 				this.MaximizeBox = false;
 				this.MinimizeBox = false;
 				this.Name = "PublishOnlineForm";
 				this.ShowIcon = false;
 				this.ShowInTaskbar = false;
 				this.SizeGripStyle = System.Windows.Forms.SizeGripStyle.Hide;
 				this.StartPosition = System.Windows.Forms.FormStartPosition.CenterScreen;
 				this.Text = "Publish Online";
 				((System.ComponentModel.ISupportInitialize)(this.pictureBoxIcon)).EndInit();
 				this.ResumeLayout(false);
 				this.PerformLayout();
 
 			}
 
 
 			private System.Windows.Forms.TextBox textBoxTitle;
 			private System.Windows.Forms.Label labelInstructions;
 			private System.Windows.Forms.Label labelTitle;
 			private System.Windows.Forms.Label labelDescription;
 			private System.Windows.Forms.TextBox textBoxDescription;
 			private System.Windows.Forms.Label labelAuthor;
 			private System.Windows.Forms.TextBox textBoxAuthor;
 			private System.Windows.Forms.Label labelDuration;
 			private System.Windows.Forms.PictureBox pictureBoxIcon;
 			private System.Windows.Forms.Button buttonOK;
 			private System.Windows.Forms.Button buttonCancel;
 			private System.Windows.Forms.ComboBox comboBoxDuration;
 			private System.Windows.Forms.CheckBox checkBoxViewPublishedDocument;
 		}
 	}
 }
 '@
 if (-not ('Addon.PublishOnline.PublishOnlineForm' -as [System.Type])) {
 	Add-Type -ReferencedAssemblies 'System.Windows.Forms','System.Drawing' -TypeDefinition $publishOnlineDialogCode
 }
 
 
 
 if (-not ('Addon.PublishOnline.WebBrowserWindow' -as [System.Type])) {
 	$cSharpCode = @'
 using System;
 using System.Linq;
 using System.Windows.Forms;
 using ActiproSoftware.UIStudio.Dock;
 using Quest.PowerGUI.SDK;
 using ToolWindow = Quest.PowerGUI.SDK.ToolWindow;
 
 namespace Addon.PublishOnline
 {
     public class WebBrowserWindow
     {
         private const string PoshCodeKey = "PoshCode";
 
         private readonly ToolWindow _poshWindow;
 
         public WebBrowserWindow()
         {
             var editor = ScriptEditorFactory.CurrentInstance;
             _poshWindow = editor.ToolWindows[PoshCodeKey] ?? editor.ToolWindows.Add(PoshCodeKey);
             _poshWindow.Visible = false;
             _poshWindow.Title = PoshCodeKey;
             if (!(_poshWindow.Control is WebBrowser))
             {
                 var webBrowser = new WebBrowser();
                 webBrowser.DocumentCompleted += (sender, args) => Activate();
                 _poshWindow.Control = webBrowser;
                 
             }
             editor.Invoke((Action)(() =>_poshWindow.Control.Parent.GetType().GetProperty("State").SetValue(_poshWindow.Control.Parent, ToolWindowState.TabbedDocument, null)));
         }
 
         public void Navigate(string url)
         {
             var webBrowser = _poshWindow.Control as WebBrowser;
             if (webBrowser == null)
             {
                 return;
             }            
 			webBrowser.Navigate(url);
         }
 
         public void Activate()
         {
             ScriptEditorFactory.CurrentInstance.Invoke((Action)(() =>
             {
                 _poshWindow.Visible = true;
                 var m = _poshWindow.Control.Parent.GetType() .GetMethods().First(it => it.Name == "Activate" && it.GetParameters().Count() == 1);
                 m.Invoke(_poshWindow.Control.Parent, new object[] {true});
             }));
         }
     }
 }	
 '@
 
 $refs = @(	"System.Windows.Forms",
 			"System.Core",
 			"System.Drawing",			
 			"$PGHome\SDK.dll", 
 			"$PGHome\ActiproSoftware.UIStudio.Dock.Net20.dll",
 			"$PGHome\ActiproSoftware.WinUICore.Net20.dll",
 			"$PGHome\ActiproSoftware.Shared.Net20.dll",
 			"$PGHome\ActiproSoftware.SyntaxEditor.Net20.dll")			
 
 Add-Type -ReferencedAssemblies $refs -IgnoreWarnings -TypeDefinition $cSharpCode | Out-Null	
 }
 
 
 	$webBrowserWindow = New-Object -TypeName 'Addon.PublishOnline.WebBrowserWindow'
 
 
 if (-not ($publishOnlineCommand = $se.Commands['FileCommand.PublishOnline'])) {
 	$publishOnlineCommand = New-Object Quest.PowerGUI.SDK.ItemCommand('FileCommand', 'PublishOnline')
 	$publishOnlineCommand.Text = 'P&ublish Online...'
 	$publishOnlineCommand.Image = $imageLibrary['PublishOnlineImage16']
 	$publishOnlineCommand.AddShortcut('Ctrl+U')
 	$publishOnlineCommand.ScriptBlock = {
 		function Publish-PoshCodeDocument {
 			[CmdletBinding()]
 			param(
 				[Parameter(Mandatory=$true)]
 				[ValidateNotNullOrEmpty()]
 				[System.String]
 				$Title,
 
 				[Parameter(Mandatory=$true)]
 				[ValidateNotNullOrEmpty()]
 				[System.String]
 				$Description = 'This file was uploaded by a PowerGUI Script Editor Add-on.',
 
 				[Parameter(Mandatory=$true)]
 				[ValidateSet('Day','Month','Forever')]
 				[System.String]
 				$Duration = 'Forever',
 
 				[Parameter(Mandatory=$true)]
 				[ValidateNotNullOrEmpty()]
 				[System.String]
 				$Author = 'Anonymous',
 
 				[Parameter(Mandatory=$true)]
 				[ValidateNotNullOrEmpty()]
 				[System.String]
 				$Script
 			)
 			[Reflection.Assembly]::LoadWithPartialName("System.Web") | Out-Null
 			[string]$data = $null
 			[string]$meta = $null
 
 			$meta = 'format=' + [System.Web.HttpUtility]::UrlEncode('posh')
 
 			if ($Duration -eq 'Day') {
 				$meta += '&expiry=d'
 			} elseif ($Duration -eq 'Month') {
 				$meta += '&expiry=m'
 			} else {
 				$meta += '&expiry=f'
 			}
 
 			if($Description) {
 				$meta += '&descrip=' + [System.Web.HttpUtility]::UrlEncode($Description)
 			} else {
 				$meta += '&descrip='
 			}   
 			$meta += '&poster=' + [System.Web.HttpUtility]::UrlEncode($Author)
 
 			$meta += '&paste=Send&posttitle=' + [System.Web.HttpUtility]::UrlEncode($Title)
 
 			$data = $meta + '&code2=' + [System.Web.HttpUtility]::UrlEncode($Script)
 
 			$request = [System.Net.WebRequest]::Create('http://PoshCode.org/')
 			$request.Credentials = [System.Net.CredentialCache]::DefaultCredentials
 			if($request.Proxy -ne $null) {
 				$request.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials
 			}
 			$request.ContentType = 'application/x-www-form-urlencoded'
 			$request.ContentLength = $data.Length
 			$request.Method = 'POST'
 
 			$post = New-Object System.IO.StreamWriter $request.GetRequestStream()
 			$post.Write($data)
 			$post.Flush()
 			$post.Close()
 
 			Write-Output $request.GetResponse().ResponseUri.AbsoluteUri
 			$request.Abort()
 		}
 
 		$se = [Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance
 		$document = $se.CurrentDocumentWindow
 		if ($document.Document.Lines.Count -le 1) {
 			[System.Windows.Forms.MessageBox]::Show("Your ""$($se.CurrentDocumentWindow.Title)"" document cannot be published online because it contains less than 2 lines of script.$([System.Environment]::NewLine *2)Add more content to your document and try again.","Error while publishing document",[System.Windows.Forms.MessageBoxButtons]::OK,[System.Windows.Forms.MessageBoxIcon]::Error) | Out-Null
 			return
 		}
 		if (-not $document.Document.IsSaved) {
 			$se.Commands['FileCommand.Save'].InvokeAsync()
 		}
 		if ($document.Document.IsSaved) {
 			$configPath = "${PSScriptRoot}\PublishOnline.config.xml"
 			$defaultTitle = $document.Title
 			$defaultDescription = 'This file was uploaded by a PowerGUI Script Editor Add-on.'
 			$defaultDuration = 'Forever'
 			$defaultAuthor = 'Anonymous'
 			$defaultViewPublishedDocument = $true
 			if (Test-Path -LiteralPath $configPath) {
 				$configuration = Import-Clixml -Path $configPath
 				if ($configuration.ContainsKey($document.Title)) {
 					$defaultTitle = $configuration[$document.Title].Title
 					$defaultDescription = $configuration[$document.Title].Description
 					$defaultDuration = $configuration[$document.Title].Duration
 					$defaultAuthor = $configuration[$document.Title].Author
 				} else {
 					$defaultAuthor = $configuration.Default.Author
 				}
 				$defaultViewPublishedDocument = $configuration.Default.ViewPublishedDocument
 			}
 			$publishOnlineDialog = New-Object -TypeName Addon.PublishOnline.PublishOnlineForm
 			$publishOnlineDialog.Image = $imageLibrary['PublishOnlineImage32']
 			$publishOnlineDialog.Title = $defaultTitle
 			$publishOnlineDialog.Author = $defaultAuthor
 			$publishOnlineDialog.Description = $defaultDescription
 			$publishOnlineDialog.Duration = $defaultDuration
 			$publishOnlineDialog.ViewPublishedDocument = $defaultViewPublishedDocument
 			$result = $publishOnlineDialog.ShowDialog()
 			if ($result -eq [System.Windows.Forms.DialogResult]::OK) {
 				$title = $publishOnlineDialog.Title
 				$author = $publishOnlineDialog.Author
 				$description = $publishOnlineDialog.Description
 				$duration = $publishOnlineDialog.Duration
 				$viewPublishedDocument = $publishOnlineDialog.ViewPublishedDocument
 				if (-not (Get-Variable -Name configuration)) {
 					$configuration = @{}
 				}
 				$configuration[$document.Title] = @{
 					      'Title' = $title
 					     'Author' = $author
 					'Description' = $description
 					   'Duration' = $duration
 				}
 				$configuration['Default'] = @{
 						           'Author' = $author
 					'ViewPublishedDocument' = $viewPublishedDocument
 				}
 				Export-Clixml -InputObject $configuration -Path $configPath
 				$documentUrl = Publish-PoshCodeDocument -Title $title -Description $description -Duration $duration -Author $author -Script $document.Document.Text
 				if ($documentUrl -ne 'http://poshcode.org/') {
 					Write-Host "Document $($document.Title) has been published at $documentUrl"
 					if ($viewPublishedDocument) {
 						$webBrowserWindow.Navigate($documentUrl)
 						if (-not ($poshCodeCommand = $se.Commands['GoCommand.PoshCode'])) {
 							$poshCodeCommand = New-Object Quest.PowerGUI.SDK.ItemCommand('GoCommand', 'PoshCode')
 							$poshCodeCommand.Text = 'Po&shCode'
 							$poshCodeCommand.Image = $imageLibrary['PublishOnlineImage16']
 							if ($goMenu = $se.Menus['MenuBar.Go']) {
 								$index = $goMenu.Items.Count + 1
 								if ($index -le 9) {
 									$poshCodeCommand.AddShortcut("Ctrl+${index}")
 								}
 							}
 							$poshCodeCommand.ScriptBlock = {
 								$webBrowserWindow.Activate()								
 							}
 							$se.Commands.Add($poshCodeCommand)
 							if ($goMenu -and (-not ($goMenu.Items['GoCommand.PoshCode']))) {
 								$goMenu.Items.Add($poshCodeCommand)
 							}
 						}
 						
 					}
 				}
 			}
 		}
 	}
 
 	$se.Commands.Add($publishOnlineCommand)
 }
 
 
 
 if (($fileMenu = $se.Menus['MenuBar.File']) -and
     (-not $fileMenu.Items['FileCommand.PublishOnline'])) {
 	if ($searchOnlineMenuItem = $fileMenu.Items['FileCommand.SearchOnline']) {
 		$index = $fileMenu.Items.IndexOf($searchOnlineMenuItem)
 		$fileMenu.Items.Insert($index + 1, $publishOnlineCommand)
 	} else {
 		$fileMenu.Items.Add($publishOnlineCommand)
 	}
 }
 
 
 
 $ExecutionContext.SessionState.Module.OnRemove = {
 	$se = [Quest.PowerGUI.SDK.ScriptEditorFactory]::CurrentInstance
 
 
 	if ($poshCodeWindow = $se.ToolWindows['PoshCode']) {
 		if (($goMenu = $se.Menus['MenuBar.Go']) -and
 		    ($poshCodeMenuItem = $goMenu.Items['GoCommand.PoshCode'])) {
 			$goMenu.Items.Remove($poshCodeMenuItem) | Out-Null
 		}
 		if ($poshCodeCommand = $se.Commands['GoCommand.PoshCode']) {
 			$se.Commands.Remove($poshCodeCommand) | Out-Null
 		}
 		$se.ToolWindows.Remove($poshCodeWindow) | Out-Null
 	}
 
 
 
 	if (($fileMenu = $se.Menus['MenuBar.File']) -and
 	    ($publishOnlineMenuItem = $fileMenu.Items['FileCommand.PublishOnline'])) {
 		$fileMenu.Items.Remove($publishOnlineMenuItem) | Out-Null
 	}
 
 
 
 	if ($publishOnlineCommand = $se.Commands['FileCommand.PublishOnline']) {
 		$se.Commands.Remove($publishOnlineCommand) | Out-Null
 	}
 
 }
 
 
`

