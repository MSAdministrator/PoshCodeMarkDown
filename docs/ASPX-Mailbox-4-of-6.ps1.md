---
Author: karl mitschke
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 508
Published Date: 2009-08-08t11
Archived Date: 2014-06-20t05
---

# aspx mailbox  (4 of 6) - 

## Description

this is part 4 of a 6 part mailbox creation web site.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 using System;
 using System.Collections.Generic;
 using System.Collections.ObjectModel;
 using System.Management.Automation;
 using System.Management.Automation.Runspaces;
 using System.Net.Mail;
 using System.Text.RegularExpressions;
 using System.Web.UI;
 using System.Web.UI.WebControls;
 
 public partial class MailboxConfirm : System.Web.UI.Page
 {
     protected void Page_Load(object sender, EventArgs e)
     {
         if (IsPostBack)
         {
             if (Session["AdminUser"].ToString() == "")
             {
                 Server.Transfer("MailboxTasks.aspx",false);
             }
             else
             {
                 Session["blKeepAddress"] = chkKeepAddress.Checked;
                 Session["NewAddress"] = newAddress.Text;
                 Server.Transfer("MailboxTaskResults.aspx", true);
             }
         }
 //Variable Definition
         Collection<PSObject> Results;
         string strNTUser = Request.ServerVariables["LOGON_USER"];
         string strUCNTUser = strNTUser.ToUpper();
         string strUserToCheck = strUCNTUser;
         string strUserName = "";
         string strTargName = "";
         string strTargPath = "";
         string strTask = "";
         Session["AdminID"] = "";
         Session["DirPath"] = "";
         Session["OldAddress"] = "";
         Session["NewAddress"] = "";
         Session["AdminUser"] = "";
         Session["LastName"] = "";
         Session["Agency"] = "";
         Session["DivisionOU"] = "";
         Session["blKeepAddress"] = false;
         newAddress.Visible = false;
         chkKeepAddress.Visible = false;
         chkKeepAddress.Text = "Keep old email addresses";
         strTargName = "'" + Request.Form[3] + "'";
         strTask = Request.Form[4];
         Session["Action"] = strTask;
         if (strUCNTUser.IndexOf("\\", 0) > 0)
         {
             strUserToCheck = strUCNTUser.Substring(strUCNTUser.IndexOf("\\",0)+1);
         }
         strNTUser = strUserToCheck.ToLower();
 
         Results = runposh("dsquery user -samid " + strNTUser);
         if (Results != null)
         {
             strUserName = Results[0].ToString();
         }
         Results = runposh("dsquery user -samid " + strTargName);
         if (Results != null)
         {
             if (Results.Count != 0)
             {
                 Response.Write("Your logon ID and password are correct<br /><br />");
                 strTargPath = Results[0].ToString();
                 fnGetData(strTargPath, strUserName, strUserToCheck, strNTUser, strTask, strTargName);
             }
             else
             {
                 Response.Write("We were unable to locate the object you specified (" + strTargName + "). Please check the name and try again<br /><br />");
                 fnAddBackButton();
             }
         }
     }
     void fnGetData(string strTargPath, string strUserName, string strUserToCheck, string strNTUser, string strTask, string strTargName)
     {
         string strPoshCommand = "";
         string strAgencyOU = "";
         string strDivOU = "";
         string strMBXGroup = "";
         string strUPN = "";
         int intAttaySize = 0;
 
         Boolean boolIsMemberOfAdminGroup = false;
         Boolean boolIsValidCompany = false;
 
         Collection<PSObject> Results;
 
         strPoshCommand = strTargPath + ".split(',').length";
         Results = runposh(strPoshCommand);
         intAttaySize = int.Parse(Results[0].ToString());
         strPoshCommand = "";
         for (int i = intAttaySize; i > 0; i--)
         {
             strPoshCommand = strPoshCommand + "$part" + i + ",";
         }
         strPoshCommand = strPoshCommand.Substring(0, strPoshCommand.Length - 1) + " = " + strTargPath + ".split(',')";
         Results = runposh(strPoshCommand);
         strPoshCommand = "dsget user " + strTargPath + " -company"; //This is the agency
         Results = runposh(strPoshCommand);
         if (Results[1].ToString().Trim().Length >= 3)
         {
             boolIsValidCompany = true;
             strAgencyOU = Results[1].ToString().Trim().Substring(0, 3).ToUpper();
             strMBXGroup = strAgencyOU + "MailboxAdmins";
         }
         else
         {
         	Response.Write("User " + strTargName + "'s Active Directory account does not seem to have an agency name in the Company field.<br>We will not be able to create a mailbox until this is corrected.<br>");
             boolIsValidCompany = false;
         	strAgencyOU = "";
         }
         if (boolIsValidCompany == true)
         {
             //Check to see if user is in the admin group
             strPoshCommand = "dsget user " + strUserName + " -memberof -expand";
             Results = runposh(strPoshCommand);
             for (int i = 0; i < Results.Count - 1; i++)
             {
                 Array groupcheck = Results[i].ToString().Split(',');
                 string strgroupstring = groupcheck.GetValue(0).ToString();
                 strgroupstring = strgroupstring.Substring(4);
 
                 if (strgroupstring.ToUpper() == strMBXGroup.ToUpper())
                 {
                     boolIsMemberOfAdminGroup = true;
                     break;
                 }
             if ((strgroupstring.ToLower() == "exchfulladmins") || (strgroupstring.ToLower() == "exch2k7servicedesk"))
                 {
                     boolIsMemberOfAdminGroup = true;
                     break;
                 }
             }
         }
         if (boolIsValidCompany == true)
         {
             if (boolIsMemberOfAdminGroup == false)
             {
                 Response.Write("User " + strUserToCheck.ToLower() + " is not a member of " + strMBXGroup + ". You will not be able to modify objects.<br><br>");
             }
             else
             {
                 if (strMBXGroup.Length > 0)
                 {
                     Response.Write("User " + strUserToCheck.ToLower() + " is a member of " + strMBXGroup + ".<br><br>");
                 }
                 Results = runposh("Add-PSSnapin Microsoft.Exchange.Management.PowerShell.Admin");
                 Results = runposh("get-user " + strNTUser + " |fl -property userprincipalname |out-string");
                 strUPN = Results[0].ToString().Replace("UserPrincipalName :", "").Trim();
                 switch (strTask.ToLower())
                 {
                     case "create mailbox":
                         fnCreateMailboxChecks(strTargPath, strUserName, strUserToCheck.ToLower(), strTargName, strAgencyOU);
                         break;
                     case "delete mailbox":
                         fnDeleteMailboxChecks(strTargPath, strUPN, strUserToCheck.ToLower());
                         break;
                     case "change address":
                         fnChangeAddressChecks(strTargPath, strUserToCheck.ToLower());
                         break;
                 }
             }
         }
     }
     void fnCreateMailboxChecks(string strUserPath, string strAdminUser, string strAdminName, string strTargName, string strAgencyOU)
     {
         Collection<PSObject> Results;
         string strCompany = strAgencyOU;
         string strLastName = "";
         string strExchangeServer = "";
 
         Response.Write("Requested action is Create Mailbox");
         Response.Write("<table border>");
         Response.Write("<tr><th>Attribute</th><th>Value</th><th>Expected State</th></tr>");
         Results = runposh("dsget user " + strUserPath + " -samid");
         Response.Write("<tr><td>Logon ID</td><td><b>" + Results[1].ToString() + "</b></td><td>Populated</td></tr>");
         Results = runposh("dsget user " + strUserPath + " -display");
         Response.Write("<tr><td>Name</td><td><b>" + Results[1].ToString() + "</b></td><td>Populated</td></tr>");
         Response.Write("<tr><td>Directory Path</td><td><b>" + strUserPath + "</b></td><td>Populated</td></tr>");
         Results = runposh("dsget user " + strUserPath + " -ln");
         strLastName = Results[1].ToString();
         Response.Write("<tr><td>Last Name</td><td><b>" + strLastName + "</b></td><td>Populated</td></tr>");
         Results = runposh("dsget user " + strUserPath + " -company");
         strCompany = Results[1].ToString();
         Results = runposh("dsget user " + strUserPath + " -upn");
         Response.Write("<tr><td>Agency</td><td><b>" + strCompany + "</b></td><td>Populated</td></tr>");
         Results = runposh("get-mailbox -identity " + strUserPath + " |fl servername |out-string");
         if (Results[0].ToString() != "")
         {
             strExchangeServer = Results[0].ToString().TrimEnd(Convert.ToChar(32)).TrimStart(Convert.ToChar(32));
             strExchangeServer = strExchangeServer.Substring(17);
             Response.Write("<tr><td>Exchange Server</td><td><b>" + strExchangeServer + "</b></td><td>Empty</td></tr></table>");
         }
         else
         {
             Response.Write("<tr><td>Exchange Server</td><td><b></b></td><td>Empty</td></tr></table>");
         }
         if ((strCompany.Replace(" ","") == "") || (strLastName.Replace(" ","") == "") || ((strExchangeServer.Replace(" ","") != "")))
         {
             Response.Write("<span style='color:Red'>One or more required fields is in an incorrect state. Please check the values listed above and try again.</span>");
             fnAddBackButton();
         }
         else
         {
             Session["AdminID"] = strAdminUser;
             Session["AdminUser"] = strAdminName;
             Session["LastName"] = strLastName;
             Session["DirPath"] = strUserPath;
             Session["Agency"] = strCompany;
             Button btnCreate = new Button();
             btnCreate.Attributes.Add("onclick", "this.value='Please wait...';this.disabled= true;" + ClientScript.GetPostBackEventReference(new PostBackOptions(this, "btnCreate")));
             btnCreate.Text = "Create Mailbox";
             frmMailboxConfirm.Controls.Add(btnCreate);
         }
     }
     void fnDeleteMailboxChecks(string strTargPath, string strAdminUser, string strAdminName)
     {
         Collection<PSObject> colDeleteResults;
 
         string strExchangeServer;
 
         Session["AdminID"] = strAdminUser;
         Session["AdminUser"] = strAdminName;
         Session["DirPath"] = strTargPath;
 
         Response.Write("Requested action is DELETE Mailbox<br>");
         Response.Write("<table border>");
         Response.Write("<tr><th>Attribute</th><th>Value</th><th>Expected State</th></tr>");
         colDeleteResults = runposh("dsget user " + strTargPath + " -samid");
         Response.Write("<tr><td>Logon ID</td><td><b>" + colDeleteResults[1].ToString() + "</b></td><td>Populated</td></tr>");
         colDeleteResults = runposh("dsget user " + strTargPath + " -display");
         Response.Write("<tr><td>Name</td><td><b>" + colDeleteResults[1].ToString() +  "</b></td><td>Populated</td></tr>");
         Response.Write("<tr><td>Directory Path</td><td><b>" + strTargPath + "</b></td><td>Populated</td></tr>");
         colDeleteResults = runposh("get-mailbox -identity " + strTargPath + " |fl servername |out-string");
         if (colDeleteResults[0].ToString() != "")
         {
             strExchangeServer = colDeleteResults[0].ToString().TrimEnd(Convert.ToChar(32)).TrimStart(Convert.ToChar(32));
             strExchangeServer = strExchangeServer.Substring(17);
 
             Response.Write("<tr><td>Exchange Server</td><td><b>" + strExchangeServer + "</b></td><td>Populated</td></tr></table>");
             Button btnDelete = new Button();
             btnDelete.Attributes.Add("onclick", "this.value='Please wait...';this.disabled= true;" + ClientScript.GetPostBackEventReference(new PostBackOptions(this, "btnDelete")));
             btnDelete.Text = "!DELETE Mailbox!";
             frmMailboxConfirm.Controls.Add(btnDelete);
         }
         else
         {
             Response.Write("<tr><td>Exchange Server</td><td><b></b></td><td>Populated</td></tr></table>");
             Response.Write("<br /><span style='color:Red'>One or more required fields is in an incorrect state. Please check the values listed above and try again.</span>");
             fnAddBackButton();
         }
     }
     void fnChangeAddressChecks(string strTargetPath, string strAdminName)
     {
         Collection<PSObject> colChangeResults;
 
         string strCompany = "";
         string strLastName = "";
         string strEmailAddress = "";
         Session["DirPath"] = strTargetPath;
 
         Response.Write("Requested action is Change Address");
         Response.Write("<table border>");
         Response.Write("<tr><th>Attribute</th><th>Value</th><th>Expected State</th></tr>");
         colChangeResults = runposh("dsget user " + strTargetPath + " -samid");
         Response.Write("<tr><td>Logon ID</td><td><b>" + colChangeResults[1].ToString() + "</b></td><td>Populated</td></tr>");
         colChangeResults = runposh("dsget user " + strTargetPath + " -display");
         Response.Write("<tr><td>Name</td><td><b>" + colChangeResults[1].ToString() + "</b></td><td>Populated</td></tr>");
         Response.Write("<tr><td>Directory Path</td><td><b>" + strTargetPath + "</b></td><td>Populated</td></tr>");
         colChangeResults = runposh("get-mailbox -identity " + strTargetPath + " |select PrimarySmtpAddress");
         if (colChangeResults.Count != 0)
         {
             strEmailAddress = colChangeResults[0].ToString();
             strEmailAddress = strEmailAddress.Substring(20);
             strEmailAddress = strEmailAddress.Substring(1, strEmailAddress.Length - 2);
             Session["AdminUser"] = strAdminName;
             Response.Write("<tr><td>Current Adress</td><td><b>" + strEmailAddress + "</b></td><td>Populated</td></tr></table>");
         }
         else
         {
             Response.Write("<tr><td>Current Adress</td><td><b></b></td><td>Populated</td></tr></table>");
         }
 
         Label lblNewAddress = new Label();
         lblNewAddress.Text = "New Address: ";
         lblNewAddress.Style["Position"] = "Absolute";
         lblNewAddress.Style["Top"] = "290px";
         lblNewAddress.Style["Left"]= "10px";
         frmMailboxConfirm.Controls.Add(lblNewAddress);
         newAddress.Style["Position"] = "Absolute";
         newAddress.Style["Top"] = "290px";
         newAddress.Style["Left"] = "100px";
         newAddress.Visible = true;
         Label lblDomain = new Label();
         lblDomain.Text = "@domain.com";
         lblDomain.Style["Position"] = "Absolute";
         lblDomain.Style["Top"] = "290px";
         lblDomain.Style["Left"] = "270px";
         frmMailboxConfirm.Controls.Add(lblDomain);
         chkKeepAddress.Style["Position"] = "Absolute";
         chkKeepAddress.Style["Top"] = "290px";
         chkKeepAddress.Style["Left"] = "385px";
         chkKeepAddress.Visible = true;
         colChangeResults = runposh("dsget user " + strTargetPath + " -company");
         strCompany = colChangeResults[1].ToString();
         colChangeResults = runposh("dsget user " + strTargetPath + " -ln");
         strLastName = colChangeResults[1].ToString();
 
         if ((strCompany.Replace(" ", "") == "") || (strLastName.Replace(" ", "") == "") || ((strEmailAddress == "")))
         {
             Response.Write("<span style='color:Red'>One or more required fields is in an incorrect state. Please check the values listed above and try again.</span>");
             newAddress.Visible = false;
             lblNewAddress.Visible = false;
             lblDomain.Visible = false;
             fnAddBackButton();
         }
         else
         {
             Session["AdminUser"] = strAdminName;
             Session["LastName"] = strLastName;
             Session["Agency"] = strCompany;
             Session["OldAddress"] = strEmailAddress;
             Session["Action"] = "Change Address";
             Session["AdminID"] = strAdminName;
             Button btnChange = new Button();
             btnChange.Attributes.Add("onclick", "this.value='Please wait...';this.disabled= true;" + ClientScript.GetPostBackEventReference(new PostBackOptions(this, "btnChange")));
             btnChange.Text = "Change Address";
             btnChange.Style["Position"] = "Absolute";
             btnChange.Style["Top"] = "325px";
             btnChange.Style["Left"] = "10px";
             frmMailboxConfirm.Controls.Add(btnChange);
         }
     }
     void fnAddBackButton()
     {
         Session["AdminUser"] = "";
         Button btnBack = new Button();
         btnBack.Attributes.Add("onclick", "this.value='Please wait...';this.disabled= true;" + ClientScript.GetPostBackEventReference(new PostBackOptions(this, "btnBack")));
         btnBack.Text = "Return";
         frmMailboxConfirm.Controls.Add(btnBack);
     }
     protected Collection<PSObject> runposh(string strCommand)
     {
         Runspace rs = GetRunspace();
             Pipeline currentPipeline = GetPipeline(rs, strCommand);
             if (currentPipeline.PipelineStateInfo.State == PipelineState.NotStarted)
             {
                 Collection<PSObject> results = currentPipeline.Invoke();
                 currentPipeline.Dispose();
                 Cache.Remove("currentPipe");
                 return (results);
             }
         else
         {
             return null;
         }
     }
     protected Runspace GetRunspace()
     {
         if (Cache["rs"] == null)
         {
             Runspace rs = RunspaceFactory.CreateRunspace();
             rs.Open();
             Cache["rs"] = rs;
         }
         return (Runspace)Cache["rs"];
     }
     protected Pipeline GetPipeline(Runspace rs, string strCommand)
     {
         if (Cache["currentPipe"] == null)
         {
             Pipeline currentPipeline = rs.CreatePipeline(strCommand);
             Cache["currentPipe"] = currentPipeline;
         }
         return (Pipeline)Cache["currentPipe"];
 
     }
 }
`

