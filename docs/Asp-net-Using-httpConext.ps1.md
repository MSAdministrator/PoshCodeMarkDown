---
Author: ti4funcom
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2443
Published Date: 
Archived Date: 2011-01-10t08
---

# asp.net-using httpconext - 

## Description

asp.net – using httpcontext

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
  Default.aspx
 ----------------
 
 
 Partial Class _Default
 Inherits System.Web.UI.Page
 
 Dim var1 As String
 Dim var2 As String
 
 Protected Sub lnk_Click(ByVal sender As Object, ByVal e As System.EventArgs) Handles lnk.Click
 Context.Items("Nome") = var1
 Context.Items("Email") = var2
 
 Server.Transfer("pagina3.aspx")
 End Sub
 
 Protected Sub Page_Load(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.Load
 var1 = "20091005"
 var2 = "20091106"
 End Sub
 End Class
 
 
 pagina3.aspx
 --------------------------
 
 Partial Class pagina3
 Inherits System.Web.UI.Page
 Protected Sub Page_Load(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.Load
 Dim context As HttpContext = HttpContext.Current
 
 Label1.Text = CStr(context.Items("Nome"))
 Label2.Text = CStr(context.Items("Email"))
 End Sub
 End Class
`

