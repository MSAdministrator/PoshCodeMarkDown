---
Author: tysonkopczynski
Publisher: 
Copyright: 
Email: 
Version: 1.0
Encoding: ascii
License: cc0
PoshCode ID: 951
Published Date: 2009-03-15t04
Archived Date: 2016-10-09t05
---

# send-htmlformattedemail - 

## Description

use this function to send an html formatted email that is based on an xslt template.  this function is based on a blog post by erik mccarty (http

## Comments



## Usage

used to send an html formatted email that is based on an xslt template.

## TODO



## function

`send-htmlformattedemail`

## Code

`#
 #
 #-------------------------------------------------
 #-------------------------------------------------
 #-------------------------------------------------
 function Send-HTMLFormattedEmail{
     param ( [string] $To,
             [string] $ToDisName,
             [string] $BCC,
             [string] $From,
             [string] $FromDisName,
             [string] $Subject,
             [string] $Content,
             [string] $Relay,
             [string] $XSLPath)
     
     try {
         $XSLArg = New-Object System.Xml.Xsl.XsltArgumentList
         $XSLArg.Clear() 
         $XSLArg.AddParam("To", $Null, $ToDisName)
         $XSLArg.AddParam("Content", $Null, $Content)
 
         $BaseXMLDoc = New-Object System.Xml.XmlDocument
         $BaseXMLDoc.LoadXml("<root/>")
 
         $XSLTrans = New-Object System.Xml.Xsl.XslCompiledTransform
         $XSLTrans.Load($XSLPath)
 
         $FinalXMLDoc = New-Object System.Xml.XmlDocument
         $MemStream = New-Object System.IO.MemoryStream
      
         $XMLWriter = [System.Xml.XmlWriter]::Create($MemStream)
         $XSLTrans.Transform($BaseXMLDoc, $XSLArg, $XMLWriter)
 
         $XMLWriter.Flush()
         $MemStream.Position = 0
      
         $FinalXMLDoc.Load($MemStream) 
         $Body = $FinalXMLDoc.Get_OuterXML()
 
         $MessFrom = New-Object System.Net.Mail.MailAddress $From, $FromDisName
         $MessTo = New-Object System.Net.Mail.Mailaddress $To
         $MessBCC = New-Object System.Net.Mail.Mailaddress $BCC
         $Message = New-Object System.Net.Mail.MailMessage $MessFrom, $MessTo
         
         $Message.Subject = $Subject
         $Message.Body    = $Body
         $Message.IsBodyHTML = $True
 
         $Message.BCC.Add($MessBCC)
      
         $Client = New-Object System.Net.Mail.SmtpClient $Relay
 
         $Client.Send($Message)
         }  
     catch {
 	return $Error[0]
         }   
     }
 
 <?xml version="1.0"?>
 <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  
 <xsl:output media-type="xml" omit-xml-declaration="yes" />
     <xsl:param name="To"/>
     <xsl:param name="Content"/>
     <xsl:template match="/">
         <html>
             <head>
                 <title>My First Formatted Email</title>
             </head>
             <body>
             <div width="400px">
                 <p>Dear <xsl:value-of select="$To" />,</p>
                 <p></p>
                 <p><xsl:value-of select="$Content" /></p>
                 <p></p>
 				<p><strong>Please do not respond to this email!</strong><br />
 					An automated system sent this email, if any point you have any questions or concerns please open a help desk ticket.</p>
 				<p></p>
             <Address>
 			Many thanks from your:<br />	
             Really Cool IT Team<br />
             </Address>
         </div>
       </body>
     </html>
     </xsl:template> 
 </xsl:stylesheet>
`

