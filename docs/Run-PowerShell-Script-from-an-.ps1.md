---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 402
Published Date: 
Archived Date: 2014-05-25t04
---

#  - 

## Description

run powershell script from an asp.net web page

## Comments



## Usage



## TODO



## script

``

## Code

`#
 #
 <%@ Page language="c#" AutoEventWireup="true" Debug="true" %>
 <%@ Import Namespace="System" %>
 <%@ Import Namespace="System.IO" %>
 <%@ Import Namespace="System.Management.Automation.Runspaces" %>
 <%@ Import Namespace="System.Management.Automation" %>
 <%@ Import Namespace="System.Collections.ObjectModel" %>
 
 
 <script language="C#" runat="server">
 
 // The previous lines use <%...%> to indicate script code, and they specify the namespaces to import. As mentioned earlier, the assemblies must be located in the \Bin subdirectory of the application's starting point.
 // http://msdn.microsoft.com/en-us/library/aa309354(VS.71).aspx
 
 //
 // Description:
 //  Run PowerShell Script from an ASP.Net web page
 //
 // Author: 
 //  Wayne Martin, 15/05/2008, http://waynes-world-it.blogspot.com/
 //
 
 private void Button3_Click(object sender, System.EventArgs e)
 {
   String fp = Server.MapPath(".") + "\\" + tPowerShellScriptName.Text;
   StreamReader sr = new StreamReader(fp);
   tPowerShellScriptCode.Text = sr.ReadToEnd();
   sr.Close();
 }
 
 private void Button2_Click(object sender, System.EventArgs e)
 {
   tPowerShellScriptResult.Text = RunScript(tPowerShellScriptCode.Text);
 }
 
 // http://msdn.microsoft.com/en-us/library/ms714635(VS.85).aspx
 
 private string RunScript(string scriptText)
 {
   Runspace runspace = RunspaceFactory.CreateRunspace();
   runspace.Open();
   Pipeline pipeline = runspace.CreatePipeline();
 
   // Create a new runspaces.command object of type script
   Command cmdScript = new Command(scriptText, true, false);
   cmdScript.Parameters.Add("-t", txtInput.Text);
   pipeline.Commands.Add(cmdScript);
     //You could also use: pipeline.Commands.AddScript(scriptText);
 
     // Re-format all output to strings
   pipeline.Commands.Add("Out-String");
 
     // Invoke the pipeline
   Collection<PSObject> results = pipeline.Invoke();
 
     //String sresults = pipeline.Output.Count.ToString();
     //sresults = sresults + "," + results.Count.ToString();
   String sresults = "";
 
   foreach (PSObject obj in results)
   {
     sresults = sresults + obj.ToString();
   }
 
   // close the runspace and set to null
   runspace.Close();
   runspace = null;
 
   return sresults;
 }
 
 
 </script> 
 
 <form id="Form1" method="post" runat="server">
 <P> <asp:Label id="Label1" runat="server" Width="104px">Parameter:</asp:Label> 
 <asp:TextBox id="txtInput" runat="server"></asp:TextBox></P>
 <P> <asp:Button id="Button3" runat="server" Text="Load" OnClick="Button3_Click"></asp:Button> </P>
 <P> <asp:Button id="Button2" runat="server" Text="Run" OnClick="Button2_Click"></asp:Button> </P>
 <P> <asp:Label id="Label2" runat="server" >Relative script name:</asp:Label> 
 <asp:TextBox id="tPowerShellScriptName" Text="test.ps1" runat="server"></asp:TextBox></P>
 <P> <asp:TextBox rows="20" columns="120" TextMode="multiline" id="tPowerShellScriptCode" runat="server"></asp:TextBox></P>
 <P> <asp:TextBox rows="8" columns="120" TextMode="multiline" id="tPowerShellScriptResult" runat="server"></asp:TextBox></P>
 </form>
`

