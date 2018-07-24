---
Author: kaktuzzz
Publisher: 
Copyright: 
Email: 
Version: 1.00
Encoding: ascii
License: cc0
PoshCode ID: 3989
Published Date: 2013-02-28t12
Archived Date: 2013-03-03t01
---

# fifteen - 

## Description

this is a fifteen game on vb.net. for russian students only;)!

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 Imports System
 Imports System.Drawing
 Imports System.Reflection
 Imports System.Globalization
 Imports System.Windows.Forms
 
 <assembly: AssemblyDescription("Fifteen")>
 <assembly: AssemblyProduct("fifteen.exe")>
 <assembly: AssemblyTitle("Fifteen")>
 <assembly: AssemblyVersion("2.0.0.0")>
 <assembly: CLSCompliant(True)>
 
 Namespace Fifteen
   Class AssemblyInfo
     Private asm As Type
 
     Public Sub New()
       asm = GetType(frmMain)
     End Sub
 
     Public ReadOnly Property Title() As String
       Get
         Dim atr As Type = GetType(AssemblyTitleAttribute)
         Dim r() As Object = asm.Assembly.GetCustomAttributes(atr, False)
         Dim ata As AssemblyTitleAttribute = CType(r(0), AssemblyTitleAttribute)
         Return ata.Title
       End Get
     End Property
   End Class
 
   Class frmMain
       Inherits Form
     Public Sub New()
       InitializeComponent()
     End Sub
 
     Private mnuMain As MainMenu
     Private mnuFile As MenuItem
     Private mnuPlay As MenuItem
     Private mnuExit As MenuItem
     Private mnuHelp As MenuItem
     Private mnuInfo As MenuItem
     Private lblTime As Label
     Private tmrTick As Timer
     Private sbMoves As StatusBar
 
     'two static fields
     Private Const pnt As Integer = 4
     Private Const nul As Integer = pnt * pnt
     'additional fields
     Private nulX As Integer = pnt - 1, nulY As Integer = pnt - 1
     Private btnNums As Button(,) = New Button(3, 3) {}
     Private num As Integer(,) = New Integer(3, 3) {}
     Private bln As Boolean
     Private mov As UInteger
     Private spn As TimeSpan
     Private rnd As New Random()
 
     Private Sub InitializeComponent()
       Me.mnuMain = New MainMenu()
       Me.mnuFile = New MenuItem()
       Me.mnuPlay = New MenuItem()
       Me.mnuExit = New MenuItem()
       Me.mnuHelp = New MenuItem()
       Me.mnuInfo = New MenuItem()
       Me.lblTime = New Label()
       Me.tmrTick = New Timer()
       Me.sbMoves = New StatusBar()
       '
       'mnuMain
       '
       Me.mnuMain.MenuItems.AddRange(New MenuItem() {Me.mnuFile, Me.mnuHelp})
       '
       'mnuFile
       '
       Me.mnuFile.MenuItems.AddRange(New MenuItem() {Me.mnuPlay, Me.mnuExit})
       Me.mnuFile.Text = "&Game"
       '
       'mnuPlay
       '
       Me.mnuPlay.Shortcut = Shortcut.F5
       Me.mnuPlay.Text = "&Play"
       AddHandler mnuPlay.Click, AddressOf mnuPlay_Click
       '
       'mnuExit
       '
       Me.mnuExit.Shortcut = Shortcut.CtrlX
       Me.mnuExit.Text = "E&xit"
       AddHandler mnuExit.Click, AddressOf mnuExit_Click
       '
       'mnuHelp
       '
       Me.mnuHelp.MenuItems.AddRange(New MenuItem() {Me.mnuInfo})
       Me.mnuHelp.Text = "&Help"
       '
       'mnuInfo
       '
       Me.mnuInfo.Text = "About..."
       AddHandler mnuInfo.Click, AddressOf mnuInfo_Click
       '
       'lblTime
       '
       Me.lblTime.BackColor = Color.Linen
       Me.lblTime.BorderStyle = BorderStyle.FixedSingle
       Me.lblTime.Font = New Font("Tahoma", 14, FontStyle.Bold)
       Me.lblTime.Location = New Point(10, 10)
       Me.lblTime.Size = New Size(pnt * 50, 20)
       Me.lblTime.Text = "00:00:00"
       Me.lblTime.TextAlign = ContentAlignment.MiddleCenter
       '
       'tmrTick
       '
       Me.tmrTick.Enabled = False
       Me.tmrTick.Interval = 1000
       AddHandler tmrTick.Tick, AddressOf tmrTick_Tick
       '
       'frmMain
       '
       Me.BackColor = Color.MintCream
       Me.ClientSize = New Size(pnt * 50 + 20, pnt * 50 + 70)
       Me.Controls.AddRange(New Control() {Me.lblTime, Me.sbMoves})
       Me.FormBorderStyle = FormBorderStyle.FixedSingle
       Me.MaximizeBox = False
       Me.Menu = Me.mnuMain
       Me.StartPosition = FormStartPosition.CenterScreen
       AddHandler Load, AddressOf frmMain_Load
     End Sub
 
     Private Sub mnuPlay_Click(ByVal sender As Object, ByVal e As EventArgs)
       Dim i As Integer, j As Integer, vec As Integer
       Dim mix As Integer = pnt * 100
 
       For k As Integer = 0 To mix - 1
         vec = rnd.Next(4)
         'moving vector
         Select Case vec.ToString(CultureInfo.CurrentCulture)
           Case "0"
             If nulX - 1 >= 0 Then
               num(nulX, nulY) = num(nulX - 1, nulY)
               nulX -= 1
             Else
               For i = 0 To pnt - 2
                 num(i, nulY) = num(i + 1, nulY)
               Next
               nulX = pnt - 1
             End If
           Exit Select
 
           Case "1"
             If nulX + 1 < pnt Then
               num(nulX, nulY) = num(nulX + 1, nulY)
               nulX += 1
             Else
               For i = pnt - 1 To 1 Step - 1
                 num(i, nulY) = num(i - 1, nulY)
               Next
               nulX = 0
             End If
           Exit Select
 
           Case "2"
             If nulY - 1 >= 0 Then
               num(nulX, nulY) = num(nulX, nulY - 1)
               nulY -= 1
             Else
               For j = 0 To pnt - 2
                 num(nulX, j) = num(nulX, j + 1)
               Next
               nulY = pnt - 1
             End If
           Exit Select
 
           Case Else
             If nulY + 1 < pnt Then
               num(nulX, nulY) = num(nulX, nulY + 1)
               nulY += 1
             Else
               For j = pnt - 1 To 1 Step - 1
                 num(nulX, j) = num(nulX, j - 1)
               Next
               nulY = 0
             End If
           Exit Select
         End Select
 
         num(nulX, nulY) = nul
       Next
 
       'break order
       For i = 0 To pnt - 1
         For j = 0 To pnt - 1
           If num(i, j) <> nul Then
             btnNums(i, j).Text = Convert.ToString(num(i, j), CultureInfo.CurrentCulture)
           Else
             btnNums(i, j).Text = ""
           End If
         Next
       Next
 
       'flush data
       mov = 0
       spn = New TimeSpan(0, 0, 0)
       bln = True
       lblTime.Text = "00:00:00"
       sbMoves.Text = "Moves: " & mov
       tmrTick.Start()
     End Sub
 
     Private Sub mnuExit_Click(ByVal sender As Object, ByVal e As EventArgs)
       Application.Exit()
     End Sub
 
     Private Sub mnuInfo_Click(ByVal sender As Object, ByVal e As EventArgs)
       Dim frm As New frmAbout()
       frm.ShowDialog(Me)
       frm.Dispose()
     End Sub
 
     Private Sub tmrTick_Tick(ByVal sender As Object, ByVal e As EventArgs)
       spn += New TimeSpan(0, 0, 1)
       lblTime.Text = spn.ToString()
     End Sub
 
     Private Sub frmMain_Load(ByVal sender As Object, ByVal e As EventArgs)
       'window caption
       Dim ai As New AssemblyInfo()
       Me.Text = ai.Title
       'generate field of buttons
       For i As Integer = 0 To pnt - 1
         For j As Integer = 0 To pnt - 1
           btnNums(i, j) = New Button()
           btnNums(i, j).Parent = Me
           num(i, j) = i * pnt + j + 1
 
           If num(i, j) <> nul Then
             btnNums(i, j).Text = Convert.ToString(num(i, j), CultureInfo.CurrentCulture)
           End If
 
           'position and style of a button
           btnNums(i, j).Left = 10 + j * 50
           btnNums(i, j).Top = 40 + i * 50
           btnNums(i, j).Size = New Size(50, 50)
           btnNums(i, j).BackColor = Color.Linen
           btnNums(i, j).Font = New Font("Tahoma", 14, FontStyle.Bold)
           btnNums(i, j).Tag = New Point(i, j)
           'event for a button
           AddHandler btnNums(i, j).Click, AddressOf btnNumX_Click
         Next
       Next
       'moves at start
       sbMoves.Text = "Moves: " & mov
     End Sub
 
     'is winner?
     Private Function PuzzleDone() As Boolean
       Dim k As Integer = 1
       For i As Integer = 0 To pnt - 1
         For j As Integer = 0 To pnt - 1
           If num(i, j) <> k Then
             Return False
           End If
 
           k += 1
         Next
       Next
 
       Return True
     End Function
 
     'counting moves
     Private Function MovesLeft() As String
       mov += 1
       Return ("Moves: " & mov)
     End Function
 
     Private Sub btnNumX_Click(ByVal obj As Object, ByVal e As EventArgs)
       If Not bln Then
         Return
       End If
 
       Dim btn As Button = DirectCast(obj, Button)
       Dim i As Integer = CType(btn.Tag, Point).X
       Dim j As Integer = CType(btn.Tag, Point).Y
 
       'moving
       If Math.Abs(i - nulX) + Math.Abs(j - nulY) = 1 Then
         num(nulX, nulY) = num(i, j)
         btnNums(nulX, nulY).Text = btnNums(i, j).Text
         'null
         nulX = i
         nulY = j
         num(nulX, nulY) = nul
         btnNums(nulX, nulY).Text = ""
         'moves
         sbMoves.Text = MovesLeft()
       End If
 
       'wins
       If nulX = pnt - 1 AndAlso nulY = pnt - 1 Then
         If PuzzleDone() Then
           tmrTick.Stop()
         End If
       End If
     End Sub
 
     'if focus lost set pause and back
     Protected Overrides Sub OnActivated(ByVal ea As EventArgs)
       MyBase.OnActivated(ea)
       If bln Then
         tmrTick.Start()
       End If
     End Sub
 
     Protected Overrides Sub OnDeactivate(ByVal ea As EventArgs)
       MyBase.OnDeactivate(ea)
       If bln Then
         tmrTick.Stop()
       End If
     End Sub
   End Class
 
   Class frmAbout
       Inherits Form
     Public Sub New()
       InitializeComponent()
     End Sub
 
     Private pbImage As PictureBox
     Private lblName As Label
     Private lblDesc As Label
     Private btnExit As Button
 
     Private Sub InitializeComponent()
       Me.pbImage = New PictureBox()
       Me.lblName = New Label()
       Me.lblDesc = New Label()
       Me.btnExit = New Button()
       '
       'pbImage
       '
       Me.pbImage.Location = New Point(16, 16)
       Me.pbImage.Size = New Size(32, 32)
       Me.pbImage.SizeMode = PictureBoxSizeMode.StretchImage
       '
       'lblName
       '
       Me.lblName.Font = New Font("Microsoft Sans Serif", 9.5F, FontStyle.Bold)
       Me.lblName.Location = New Point(52, 19)
       Me.lblName.Size = New Size(360, 18)
       Me.lblName.Text = "Fifteen v1.00"
       '
       'lblDesc
       '
       Me.lblDesc.Location = New Point(67, 37)
       Me.lblDesc.Size = New Size(360, 23)
       Me.lblDesc.Text = "This is an example that you can make better."
       '
       'btnExit
       '
       Me.btnExit.Location = New Point(135, 67)
       Me.btnExit.Text = "OK"
       '
       'frmAbout
       '
       Me.AcceptButton = Me.btnExit
       Me.CancelButton = Me.btnExit
       Me.ClientSize = New Size(350, 100)
       Me.Controls.AddRange(New Control() {Me.pbImage, Me.lblName, Me.lblDesc, Me.btnExit})
       Me.FormBorderStyle = FormBorderStyle.FixedToolWindow
       Me.ShowInTaskbar = False
       Me.StartPosition = FormStartPosition.CenterParent
       Me.Text = " About..."
       AddHandler Load, AddressOf frmAbout_Load
     End Sub
 
     Private Sub frmAbout_Load(ByVal sender As Object, ByVal e As EventArgs)
       pbImage.Image = Me.Owner.Icon.ToBitmap()
     End Sub
   End Class
 
   Friend NotInheritable Class Program
     <STAThread()> _
     Shared Sub Main()
       Application.EnableVisualStyles()
       Application.Run(New frmMain())
     End Sub
   End Class
 End Namespace
`

