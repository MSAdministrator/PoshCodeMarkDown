---
Author: 
Publisher: 
Copyright: 
Email: 
Version: 0.1
Encoding: ascii
License: cc0
PoshCode ID: 2529
Published Date: 
Archived Date: 2011-03-02t12
---

# openrasta - 

## Description

basic authentication class, with really simple dumb cookie set.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 	public class MyBasicAuthenticator : IBasicAuthenticator
 	{
 		private readonly IUserCredentials _users;
 
 		public ICommunicationContext Context { get; set; }
 
 		public MyBasicAuthenticator(IUserCredentials users)
 		{
 			_users = users;
 		}
 
 		public string Realm
 		{
 			get { return "Web Service v1"; }
 		}
 
 		/// <summary>
 		/// Create the cookie if doesnt exist allready, really simmple at moment.
 		/// </summary>
 		private void SetCookie(string key, string val)
 		{
 			var response = Context.Response;
 			var request = Context.Request;
 
 			if (!request.Headers.ContainsKey("Cookie"))
 			{
 				if (!response.Headers.ContainsKey("Set-Cookie")) // POINT 1. not sure why this is required.
 				{
 					response.Headers.Add("Set-Cookie", String.Format("{0}={1};", key, val));
 				}
 			}
 		}
 
 		public AuthenticationResult Authenticate(BasicAuthRequestHeader header)
 		{
 			var credentials = _users.GetCredentialsFor(header.Username);
 			if (credentials != null && credentials.Password == header.Password)
 			{
 				SetCookie("sessionkey", "moo");
 				return new AuthenticationResult.Success(header.Username);
 			}
 			return new AuthenticationResult.Failed();
 		}
 	}
 
 
 // how its registered //
 				ResourceSpace.Uses.BasicAuthentication<MyBasicAuthenticator>();
 
 	public static class ExtensionsToIUses
 	{
 		public static void BasicAuthentication<TBasicAuthenticator>(this IUses uses) where TBasicAuthenticator : class, IBasicAuthenticator
 		{
 			uses.CustomDependency<IAuthenticationScheme, BasicAuthenticationScheme>(DependencyLifetime.Singleton);
 			uses.CustomDependency<IBasicAuthenticator, TBasicAuthenticator>(DependencyLifetime.Transient);
 		}
 	}
`

