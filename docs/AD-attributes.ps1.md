---
Author: paul mcdonnell
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 3383
Published Date: 2012-04-24t08
Archived Date: 2015-08-31t01
---

# ad attributes - 

## Description

this asp page takes an input from a form from a html page. the input is the displayname (or part of a displayname) of an account in active directory. it returns a number of fields, but the one i was mainly interested in was the msds-principalname, which is only available in ad 2008. as we’re not 100% ad 2008, i hard-coded this page to hit a windows 2008 domain controller. i also needed the msds-principalname field with “winnt

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <%@ Language=VBScript %>
 <% StartTime = Timer %>
 <html>
 <head><title><%=strCompany%> - Search List</title>
 </head>
 <body>
 <link rel="stylesheet" href="/server/default.css" TYPE="text/css" media="screen">
 <%
 Dim MyVariable
 MyVariable=Request.Form("name")
 MyVariable =Replace(MyVariable,"'","''")  
     'the request.form gets the info within the boxes of the form actioned
     'from the previous HTML/ASP
     gotname = Request.form("name")
     %> 
 
 
 <%
 Dim objRootDSE, objConnection, objRecordSet, objCommand
 Dim strDomainLDAP, intPage, i, j, strTRbgColor, strTemp
 Const ADS_SCOPE_SUBTREE = 2
 
 Set objRootDSE = GetObject("GC://RootDSE")
 strDomainLDAP = objRootDSE.Get("DefaultNamingContext")
 Set objRootDSE = Nothing
 
 response.write("Display Name that was searched for: <h3>" & gotname)
 
 
 Set objConnection = CreateObject("ADODB.Connection")
 Set objRecordSet = Server.CreateObject("ADODB.Recordset")
 Set objCommand = Server.CreateObject("ADODB.Command")
 
 objConnection.Provider = "ADsDSOObject"
 objConnection.Open "Active Directory Provider"
 
 Set objCommand.ActiveConnection = objConnection
 objCommand.Properties("Page Size") = 10000
 objCommand.Properties("Searchscope") = ADS_SCOPE_SUBTREE
 objCommand.Properties("Size Limit") = 10000
 
 objCommand.CommandText = "SELECT mail, l, title, Displayname, msds-PrincipalName" & _
 			" FROM 'GC://windows2008DC' WHERE objectCategory='user' AND homemdb='*' AND UserAccountControl='512' AND objectclass='User' AND displayname= '*" & myvariable & "*' ORDER BY Name"
 
 objRecordSet.Open objCommand.CommandText, objConnection,1,1
 objRecordSet.PageSize = Cint(RowsPerPage)
 
 If Request.QueryString("page") = "" Then
 	intPage = 1
 Else
 	intPage = Request.QueryString("page")
 End If
 
 If objRecordSet.EOF Then 
 	Response.Write "<h3>No search term entered or search term not found</h3>" & vbCrLf
 	Response.End 
 Else
 	Response.Write "<center><h3>" & strCompany & " - Searchlist </h3>" & vbCrLf
 	
 	objRecordSet.AbsolutePage = intPage
 
 	Response.Write "<table align=center border=1>" & vbNewLine
 	Response.Write "	<tr align=center>" & vbNewLine
 	For i = 0 To objRecordSet.Fields.Count - 1
 		Response.Write "		<th>" & objRecordSet.Fields(i).Name & "</th>" & vbNewLine
 	Next
 	Response.Write "	</tr>" & vbNewLine
 rem Response.Write objRecordSet.RecordCount
 	For j = 1 To objRecordSet.RecordCount
 rem objRecordSet.PageSize
 		Response.Write "	<tr bgColor='"
 		strTRbgColor = RowOddColor
 		If j Mod 2 = 0 Then strTRbgColor = RowEvenColor
 		Response.Write strTRbgColor 
     	If RowMoveColor <> "" Then
     		Response.Write "' onmouseover=this.bgColor='" & RowMoveColor & "' onmouseout=this.bgColor='" & strTRbgColor
     	End If
     	Response.Write "'>" & vbCrLf		
 			If objRecordSet.Fields(0).Value <> "" Then strTemp = objRecordSet.Fields(0).Value
 			Response.Write "		<td>winnt://" & strTemp & "</td>" & vbNewLine
 			For i = 1 To objRecordSet.Fields.Count - 1
 			strTemp = "<font color=" & strTRbgColor & ">-</font>"
 			If objRecordSet.Fields(i).Value <> "" Then strTemp = objRecordSet.Fields(i).Value
 			Response.Write "		<td>" & strTemp & "</td>" & vbNewLine
 
 		Next						
 
 		Response.Write "	</tr>" & vbNewLine
 	    objRecordSet.MoveNext
 	Next
 
 	Response.Write "</table>" & vbNewLine
 
 	Response.Write "<br><br>" & vbNewLine	
 
 
 objRecordSet.Close
 Set objRecordSet = Nothing
 Set objConnection = Nothing
 End If
 %>
 </body>
 </html>
 
 <%
 Private Function TrimServerVariable(strString)
 Dim temp 'As String
 If inStr(strString,"&") <> 0 Then
 	arr = Split(strString, "&")
 	strString = ""
 	For i = 0 To UBound(arr) - 1
         temp = Left(arr(i), InStr(arr(i), "="))
 		If InStr(strString, temp) = 0 Then
 		strString = strString & arr(i) & "&"
 		End If
 	Next
 	strString = Left(strString, Len(strString) - 1)
 End If
 TrimServerVariable = strString
 End Function
 %>
 <%
 EndTime = Timer
 Response.Write "<!-- Page rendered in " & (EndTime-StartTime) & " -->" & vbNewLine
 %>
`

