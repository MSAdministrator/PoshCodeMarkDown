---
Author: joel bennett
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 69
Published Date: 2008-12-09t11
Archived Date: 2017-04-30t12
---

# get-cert - 

## Description

a script to retrieve the ssl certificate used by a remote host … demonstrates using invoke-inline to compile c# code, and handling the remotecertificatevalidationcallback to override the normal security policy …

## Comments



## Usage



## TODO



## 

``

## Code

`#
 #
 $UsingStatements = @"
 using System.Net.Security;
 using System.Net.Sockets;
 using System.Security.Authentication;
 using System.Security.Cryptography.X509Certificates;
 "@
 
 $GetCert = @"
 RemoteCertificateValidationCallback callback = delegate(
 	object sender, 
 	X509Certificate cert,
 	X509Chain chain, 
 	SslPolicyErrors sslError)
 {
 	X509Certificate2 x509 = new X509Certificate2(cert);
 	result.Add(x509);
 
 	// Print to console information contained in the certificate.
 	Console.WriteLine("Subject: {0}", x509.Subject);
 	Console.WriteLine("Issuer: {0}", x509.Issuer);
 	Console.WriteLine("Version: {0}", x509.Version);
 	Console.WriteLine("Valid Date: {0}", x509.NotBefore);
 	Console.WriteLine("Expiry Date: {0}", x509.NotAfter);
 	Console.WriteLine("Thumbprint: {0}", x509.Thumbprint);
 	Console.WriteLine("Serial Number: {0}", x509.SerialNumber);
 	Console.WriteLine("Friendly Name: {0}", x509.PublicKey.Oid.FriendlyName);
 	Console.WriteLine("Public Key Format: {0}", x509.PublicKey.EncodedKeyValue.Format(true));
 	Console.WriteLine("Raw Data Length: {0}", x509.RawData.Length);
 //	Console.WriteLine("Certificate to string: {0}", x509.ToString(true));
 //	Console.WriteLine("Certificate to XML String: {0}", x509.PublicKey.Key.ToXmlString(false));
 
 	Console.WriteLine("Added a certificate. Total: " + result.Count );
 	
 	if (sslError != SslPolicyErrors.None) {
 		Console.WriteLine("Certificate error: " + sslError);
 	}
 		
 	return false; // always stop, we have what we need
 };
 
 foreach(string serverName in args) { 
 	Console.WriteLine("\n\nFetching SSL cert for {0}\n", serverName);
 	// int secondArg = (int) ((object[]) arg)[1]; 
 
 
 	// Create a TCP/IP client socket to a machine name
 	TcpClient client = new TcpClient(serverName,443);
 	// Create an SSL stream that will close the client's stream.
 	SslStream sslStream = new SslStream( client.GetStream(), false, callback, null );
 	
 	try 
 	{
 		sslStream.AuthenticateAsClient(serverName);
 	} 
 	catch (AuthenticationException e)
 	{
 		Console.WriteLine("Exception: {0}", e.Message);
 		if (e.InnerException != null)
 		{
 			Console.WriteLine("Inner exception: {0}", e.InnerException.Message);
 		}
 		Console.WriteLine ("Authentication failed - closing the connection.");
 	}
 	client.Close();
 }
 "@
 
 .\Invoke-Inline $UsingStatements,$GetCert $args -ref @()
`

