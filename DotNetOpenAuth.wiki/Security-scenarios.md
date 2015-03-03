# Security scenarios with ASP.NET MVC, ASP.NET Web API, OAuth2 and DotNetOpenAuth

> #### About the Author:
> This article has been contributed by Manfred Steyer. Manfred is Head of the part-time Degree Programme in Software Engineering Leadership of the Information Technologies and Business Informatics Degree Programme at CAMPUS 02, University of Applied Sciences in Graz and a freelance trainer and consultant at www.IT-Visions.de. He offers in-house trainings regarding ASP.NET MVC and ASP.NET Web API and speaks regularly at conferences about Software Development with .NET. In his current book which he is writing for Microsoft Press in Germany he describes the use of ASP.NET MVC for the development of modern web applications.

Nowadays, web users have to contend with numerous user accounts, resulting in the users' desire to access the web services of their choice via one account or a limited number of accounts. Moreover, users are increasingly looking to pass on parts of their access rights to third parties without revealing their passwords. Social networks which offer to search the user's address book for potential contacts who are also signed up to that particular social network are a good example of this.

A solution to these demands is the use of security tokens. A token can be compared to a permit, with the issuer confirming that he grants certain (access) rights to the holder of the permit. For the issuance and the use of these tokens to work smoothly between all parties concerned, they are advised to agree on a common protocol. OAuth2 is an example of such a protocol and is currently very popular for web applications.

##About OAuth

The first version of OAuth was developed in 2006 by Twitter and Ma.gnolia with the aim of giving users the ability to grant a part of their rights to third parties without sharing their own passwords. By doing so, applications can be granted the right to publish or retrieve messages in the name of a Twitter user.

Meanwhile, OAuth and/or its successor OAuth2 are used by major players such as Google, Facebook, Flickr, Microsoft, Salesforce.com and Yahoo!. Thereby it is notable that it is increasingly used not only to delegate rights (authorization) but also for single sign-on scenarios (authentication). This means that users can, for example, use their Google account to access other web solutions, whereby the web solution in question obtains the right to access the profile data of the logged-on Google user. There are also other companies of those listed above offering this very service. While some call this a misuse of OAuth2 and refer to the existing security loopholes, others are working on standardizing the corresponding procedure under the name of OpenId Connect [3], which shows how the security loopholes pointed out by critics can be closed.

OAuth2 defines four roles interacting with each other: resource owner, resource server, client and authorization server. The resource owner is the owner of a resource provided via a resource server. Upon request of the resource owner, other applications are issued a token by the authorization server enabling them to access certain resources.

In the case of Facebook, Facebook itself is the resource server while at the same time also being the authorization server. The resource owner, however, is the Facebook user who uploads resources, such as photos. The client role would then be held by a third party application being granted access to these pictures.

In addition to that, OAuth2 defines several “flows” for different use-cases (see http://tools.ietf.org/html/draft-ietf-oauth-v2-31). This document concentrates on the so called Authorization Code Flow for browser-based scenarios and on the Resource Owner Password Credentials Flow for REST-Services.

## Access to shared resources via web applications

A web application which wants to gain access to shared resources should redirect the user to a page of the authorization server. When doing so, it informs the authorization server about the access rights it is requesting. This information, which is called Scope, is actually a list of identifiers separated by space characters which are often available as URLs in order to avoid naming conflicts and are given by the resource server.

Consequently, the authorization server asks the user, in his role as resource owner, for authentication (e.g. by providing the username and password). Subsequently, the user can either grant or reject the client’s request. The authorization server then redirects the user to the client and passes the user's decision to the client using a URL parameter. If the user has granted the request, the query string contains a code which the client can exchange for a security token. When doing so, the client provides authentication details to the authorization server. Mostly this is also done by giving the username and password.

The token received this way may then be used by the client to gain access to the desired resources via the resource server. Once it has received the token, the resource server must verify its validity and check if it was indeed provided by the named authorization server. Validity can be checked using the expiry date contained within the token and the latter may be done by verifying other evidence which is also embedded in the token. Such evidence may, for example, be a digital signature or an HMAC. If such cryptographic proceedings are to be avoided, there is also the possibility of the resource server contacting the authorization server to confirm the validity of the token.

The token may contain information about the user which might be used by the resource server to verify rights. Alternatively, the token may simply be a key which the resource server may exchange for user-related data when contacting the authorization server.

## Implementation using ASP.NET MVC, DotNetOpenAuth and OAuth2

The following sections demonstrate how the scenario which has been discussed above may be realized using the free library DotNetOpenAuth. DotNetOpenAuth is an accepted .NET library enabling the implementation of the OAuth and OAuth2 protocols as well as the realization of OpenId. It may be obtained via download or NuGet. At the time when this text was written, the required NuGet package was called DotNetOpenAuth (unified). If this changes in the future, the descriptions of the packages found when searching for DotNetOpenAuth will offer the information necessary for selecting the suitable version.

### Authentication and authorization via Google

To keep it simple, the first example described below is based on an authorization server and a resource server which already exist. For these two roles, services provided by Google are used. To enable this, the developer must register the application in the Google API Console in http://code.google.com/apis/console and create access possibilities for every client (see Fig. 1).

_Fig. 1: Registration of an application and a client in the Google API Console_
![Registration of an application and a client in the Google API Console](https://raw.github.com/wiki/DotNetOpenAuth/DotNetOpenAuth/images/OAuth2_GoogleApiConsole_011.png)

The WebClient class is responsible for implementing OAuth2 within the web application. The CreateClient method in Listing 1 is an example of the instantiation of this class. It provides the constructor with an AuthorizationServerDescription type of object and the ID which was assigned to the client by Google. The latter can be learned via the API Console (this is also true for the required client secret) (see Fig. 1). The AuthorizationServerDescription contains information about the authorization server, and for the example at hand it is created using the GetAuthServerDescription method. The address used by the user for authentication is given using AuthorizationEndpoint, and the address via which the web client finally obtains the token is given using TokenEndpoint.

The CreateClient method also defines the client secret (“password”) to be used. To this end it passes the client secret to the ClientCredentialApplicator.PostParameter method and subsequently assigns the return value of this method to the client's ClientCredentialApplicator property. During this process, PostParameter creates an instance of an internal sub-class of ClientCredentialApplicator which causes the WebClient to transfer the client ID and the client secret within the HTTP payload. This is necessary due to an incompatibility between Google and the.NET classes on which DotNetOpenAuth is based. However, if the counterpart is able to use the well-known HTTP Basic schema for client authentication, the developer could alternatively introduce the ClientCredentialApplicator.NetworkCredential method, or, as HTTP Basic is employed as standard, pass the Secret to the WebClient using a third optional constructor argument.

_Listing 1_

```c#

	public class AuthHelper
	{
	       public static WebServerClient CreateClient()
	       {
	               var desc = GetAuthServerDescription();
	               var client = new WebServerClient(desc, clientIdentifier: "xyz.apps.googleusercontent.com");
	               client.ClientCredentialApplicator = ClientCredentialApplicator.PostParameter("some_password");
	               return client;
	       }
	
	       public static AuthorizationServerDescription GetAuthServerDescription()
	       {
	               var authServerDescription = new AuthorizationServerDescription();
	               authServerDescription.AuthorizationEndpoint = new Uri(@"https://accounts.google.com/o/oauth2/auth");
	               authServerDescription.TokenEndpoint =  new Uri(@"https://accounts.google.com/o/oauth2/token"); 
	               authServerDescription.ProtocolVersion = ProtocolVersion.V20;
	               return authServerDescription;
	       }
	}

```

The controller shown in Listing 2 illustrates how a WebClient instance can be used in an ASP.NET MVC project. The whole process starts with the OAuth action method which determines if it is being initiated directly by the user or retrieved by the authorization server once the user has granted access to his resources. If the latter is the case, the code parameter which can be exchanged by the web application for the required token, is being called.

In the event of a direct call by the user, OAuth delegates to InitAuth. First, this method creates an instance of AuthorizationState describing the authorization request of the client. Thereby, the method in question defines the callback URL to be used by the authorization server as well as the desired scope. As the URL which has just been called should also be used as a callback URL in this case, it finds out about the uri by calling Request.Url.AbsoluteUri and removes a potential query string using the helper method RemoveQueryStringFromUri. Thereby it must be taken into account that, for security reasons, Google only allows callback URLs which have been registered for the client in the API Console. The comparison between registered and passed callback URLs is case-sensitive (!).

The InitAuth method uses the identifiers foreseen by Google for the user's profile information, his email address and his calendar as Scope. Such information is to be found in the documentation provided by Google [4]. Subsequently, InitAuth creates an object which describes the authorization request using the PrepareRequestUserAuthorization method. By using the AsActionResult extension method, it transfers the latter to an ActionResult which it then returns. The ActionResult initiates ASP.NET MVC to route the user to the Authorization Server – as described at the beginning of this article. There, the user must identify himself and can finally grant or reject the authorization request of the web client.

Once the user has informed the authorization server about his decision, the latter routes the user to a defined callback URL. If the decision is positive, the respective call includes the URL parameter code and as a consequence, the OAuth action method delegates to OAuthCallback. OAuthCallback uses the ProcessUserAuthorization method which is provided by the WebClient to exchange this code for a token.

This token can now be used to access resources of the user which are described by the requested Scope. Therefore, the token must be transferred when the desired resources are requested via HTTP. Usually, the HTTP header Authorization is used for this purpose, as shown in the GetUserInfo method (Listing 4). This header entry consists of two parts separated by spaces: The first part represents the desired authorization schema; the second part contains the credentials, such as username and password, or a token. OAuth2 treats Bearer as the name of the authorization schema – as a matter of fact, the tokens in question are also called bearer tokens.

By definition, OAuth2 tokens are only valid for a short period of time to prevent misuse. They can, however, be refreshed by the client if necessary. Therefore, the WebClient class offers the Refresh method. The date of issuance and the date of expiry of the token are given in its properties.

_Listing 2_

```c#

using DotNetOpenAuth.OAuth2;
using DotNetOpenAuth.Messaging;

public class SecureController : Controller
{
        public ActionResult OAuth()
       {
			if (string.IsNullOrEmpty(Request.QueryString["code"]))
			{
				  return  InitAuth();
			}
			else
			{
				  return  OAuthCallback();
			}
       }

       static WebServerClient client = AuthHelper.CreateClient();

       private  ActionResult InitAuth()
       {
			var state = new AuthorizationState();
			var uri = Request.Url.AbsoluteUri;
			uri = RemoveQueryStringFromUri(uri);
			state.Callback = new Uri(uri);

			state.Scope.Add("https://www.googleapis.com/auth/userinfo.profile");
			state.Scope.Add("https://www.googleapis.com/auth/userinfo.email");
			state.Scope.Add("https://www.googleapis.com/auth/calendar");

			var r = client.PrepareRequestUserAuthorization(state);
			return r.AsActionResult();
       }

       private static string RemoveQueryStringFromUri(string uri)
       {
			int index = uri.IndexOf('?');
			if (index > -1)
			{
				  uri = uri.Substring(0, index);
			}
			return uri;
       }

       private ActionResult OAuthCallback()
       {
			var auth = client.ProcessUserAuthorization(this.Request);
			Session["auth"] = auth;

			[…]

			dynamic userInfo = google.GetUserInfo(auth.AccessToken);
			// Later, if necessary:
			// bool success = client.RefreshAuthorization(auth);
       }
}
```

## Validating tokens to close security loopholes ##

As mentioned at the beginning of this article, OAuth2 was created to authorize applications (clients) in order to grant them limited access to the resources of a user. In the example which we have just discussed, the web client has requested access to the profile and to the calendar data of the current user.

In fact, this has nothing to do with authentication. However, one could think that to authenticate a user with the implemented web client one would simply need to access his profile data as it is possible to identify a user this way. The problem is that this procedure does not make sure that the user really wanted to gain access to the implemented client. Theoretically, the authorization server could have issued the token for a completely different application, which now accesses the implemented web client in the name of the user and without his authorization. To close this potential security loophole, a client wanting to use OAuth for authorization purposes should validate the token. When doing so, the client must check if the token was actually issued for accessing the client's services.

Listing 3 shows how the token obtained from Google can be validated. First, it retrieves further information from Google about the token using the GetTokenInfo method of the GoogleProxy user-defined class. Subsequently, it passes this information to the ValidateToken method for validation. If validation has been successful, it can retrieve the data from the user profile and use it to authenticate the user. The GetUserInfo method of the GoogleProxy class is used to do this.

Listing 4 describes the implementation of the GoogleProxy class. It uses the HttpClient class which was introduced with .NET 4.5 to call REST services. Moreover, work with JSON is supported by JSON.net, a free library which can be obtained via NuGet. Both methods request the result of the initiated HTTP request from the HttpClient in form of a JObject. This is a dynamic object representing the objects transferred in JSON format.

As the token is to be used for authorization with the resource server when working with GetUserInfo, it is also transferred in the Authorization HTTP header. If GetTokenInfo is used, the token is to be found as a URL parameter in the request, as this is where further information about the token is requested.

The TokenValidator class is to be found in Listing 5. Its ValidateToken method takes over the information which has been found using GoogleProxy.GetTokenInfo and verifies its audience property. The token is only created for the current client if it bears the Id which has been selected for the current client in the Google API Console. If this is not the case, an exception is initiated. Moreover, the ValidateToken method checks the expires_in property. If its value is equal to or smaller than zero, the token is no longer valid, which also leads to an exception.

This way of checking tokens complies with Google [4] documentation and correlates with the current work-in-progress version of the future OpenId Connect standard which is based on OAuth2 and is designed for authentication scenarios similar to the one discussed here.

Once the data of the user has been stored in the session, a session cookie may be created. This may, for example, be done using the authentication forms integrated in the ASP.NET processes.

*Listing 3*

```c#
	
	var google = new GoogleProxy();
	var tv = new TokenValidator();
	
	dynamic tokenInfo = google.GetTokenInfo(auth.AccessToken);
	tv.ValidateToken(tokenInfo, expectedAudience: "xyz.apps.googleusercontent.com");
	
	dynamic userInfo = google.GetUserInfo(auth.AccessToken);
	
	Session["user.name"] = userInfo.name;
	Session["user.birthday"] = userInfo.birthday;
	Session["user.locale"] = userInfo.locale;
	Session["user.userid"] = userInfo.id;
	Session["user.email"] = userInfo.email;
```

*Listing 4*

```c#
	[…]
	[…]
	using System.Net.Http;
	using System.Net.Http.Headers;
	using DotNetOpenAuth.OAuth2;
	using Newtonsoft.Json.Linq;
	[…]
	
	public class GoogleProxy
	{
	      public dynamic GetUserInfo(string authToken)
	      {
				var userInfoUrl = "https://www.googleapis.com/oauth2/v1/userinfo"; 
				var hc = new HttpClient();
				hc.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", authToken);
				var response = hc.GetAsync(userInfoUrl).Result;
				dynamic userInfo = response.Content.ReadAsAsync().Result;
				return userInfo;
	      }
	      public dynamic GetTokenInfo(string accessToken)
	      {
				var verificationUri = 
					"https://www.googleapis.com/oauth2/v1/tokeninfo?access_token=" 
					 + accessToken;
	
				var hc = new HttpClient();
	
				var response = hc.GetAsync(verificationUri).Result;
				dynamic tokenInfo = response.Content.ReadAsAsync().Result;
	            return tokenInfo;
	      }
	
	}
```

*Listing 5*

```c#
public class TokenValidator
public class TokenValidator
{
       public void ValidateToken(dynamic tokenInfo, string expectedAudience)
       {
		   var audience = tokenInfo.audience.ToString();
		   if (string.IsNullOrEmpty(audience) || audience != expectedAudience)
		   {
				  var e = new HttpException("tokes with unexpected audience: ");
				  throw e;
		   }

           if (tokenInfo.expires_in == null) return;
           var expiresIn = tokenInfo.expires_in.ToString();
           int intExpiresIn;
           var isInt = int.TryParse(expiresIn, out intExpiresIn);

           if (!isInt || intExpiresIn

```

## Developing a user-specific authorization server ##

The previous sections have described the development of a client compatible with OAuth2. This section elaborates on implementing a user-specific OAuth2-compatible authorization server. Implementing the IAuthorizationServerHost interface which has been provided by DotNetOpenAuth is key for this process. Listing 6 includes an example showing how this task can be managed. To simplify this example, the use of a database has been avoided. Instead, static listings or hard-coded information is used.

GetClient is invoked by DotNetOpenAuth to provide information about the calling client. In the present example, there is only one client with RP as an ID. The returned value provided by GetClient is an IClientDescription instance. Apart from the client ID and the client secret, this instance also contains the permitted callback URL.

The IsAuthorizationValid method checks if the current user may delegate the requested rights (=Scope) to the client. The present implementation only allows for the user Max Muster to delegate the right described using the http://localhost/demo scope to the RP client.

CreateAccessToken is responsible for issuing the token. In addition to the expiry date, this method only defines a private signature key and a public key to encode the token. The Scope as well as the Client Id and the username are transferred to the token after validation has been performed by IsAuthorizationValid.

By means of the signature the authorization server proves that it actually is the issuer; the encoding process ensures that the resource server only is able to read the token's content. The method in question takes the two keys needed from the Windows Certificate Store. For this it uses the private helper method LoadCert which expects the fingerprint of the desired certificate. To avoid magic strings, these fingerprints are to be found in the public constants of the Config class (Listing 7).

To keep the implementation of OAuth2 simple, the use of signatures and codes is seen as optional. The signature does not have to be used as the resource server retrieves the token directly from the authorization server and also the token does not need to be encoded, as OAuth2 prescribes the use of SSL. Nevertheless, the use of these measures creates a further level of security, as it can reduce potential attack scenarios.

The CryptoStore property returns a user-defined implementation of ICryptoStore. The tasks of the latter consist of administering issued access codes and tokens. Similarly, NonceStore returns a user-defined implementation of INonceStore. A NonceStore is responsible for storing single-use passwords, so-called nonces, which are used internally by DotNetOpenAuth. As the store stores nonces which have already been used it can be verified whether a Nonce which has been created anew has already been used. An example of another simple CryptoStore which uses a static list with CryptoKeyStoreEntry instances (Listing 9) for demonstration reasons is to be found in Listing 10. In any case, this CryptoStore must be thread-safe – a prerequisite to avoid problems in unforeseen circumstances. A dummy implementation of INonceStore which is suitable for demonstration purposes is to be found in Listing 11. Instead of checking if the passed Nonce already exists and to save it subsequently, the shown StoreNonce method shows with every call that the passed Nonce has not been used before.

AutomatedAuthorizationCheckResponse throws only a NotImplementedException in the present implementation. The same is true of AutomatedUserAuthorizationCheckResponse. The reason for this is that the two methods are not used in the scenario discussed in this paper. AutomatedAuthorizationCheckResponse is required if the client needs access to the resource server in his own name and not in the name of the user. On the contrary, AutomatedUserAuthorizationCheckResponse is used if the client knows the username and the user's password. This scenario, which, according to the OAuth2 specification requires a lot of trust of the user in the client, will be discussed later on.

*Listing 6*

```c#
	public class AuthServerHostImpl : IAuthorizationServerHost
	{
	    public IClientDescription GetClient(string clientIdentifier)
	    {
	        switch (clientIdentifier)
	        {
	
	            case "RP":
	                var allowedCallback = "https://localhost/RP/Secure4ownAuthSvr/OAuth";
	                return new ClientDescription(
	                             "data!", 
	                             new Uri(allowedCallback), 
	                             ClientType.Confidential);
	        }
	
	        return null;
	    }
	
	    public bool IsAuthorizationValid(IAuthorizationDescription authorization)
	    {
	        if (authorization.ClientIdentifier == "RP" 
	              && authorization.Scope.Count == 1 
	              && authorization.Scope.First() == "http://localhost/demo" 
	              && authorization.User == "Max Muster")
	        {
	            return true;
	        }
	
	        return false;
	    }
	
	    public AccessTokenResult CreateAccessToken(
	                 IAccessTokenRequest accessTokenRequestMessage)
	    {
	
	        var token = new AuthorizationServerAccessToken();
	
	        token.Lifetime = TimeSpan.FromMinutes(10);
	
	        var signCert = LoadCert(Config.STS_CERT);
	        token.AccessTokenSigningKey = 
	                 (RSACryptoServiceProvider) signCert.PrivateKey;
	
	        var encryptCert = LoadCert(Config.SERVICE_CERT);
	        token.ResourceServerEncryptionKey = 
	                 (RSACryptoServiceProvider) encryptCert.PublicKey.Key;
	
	        var result = new AccessTokenResult(token);
	        return result;
	    }
	
	    private static X509Certificate2 LoadCert(string thumbprint)
	    {
	        X509Store store = new X509Store(StoreName.My, StoreLocation.LocalMachine);
	        store.Open(OpenFlags.ReadOnly);
	        var certs = store.Certificates.Find(
	                          X509FindType.FindByThumbprint, 
	                          thumbprint, 
	                          validOnly: false);
	
	        if (certs.Count == 0) throw new Exception("Could not find cert");
	        var cert = certs[0];
	        return cert;
	    }
	
	    public DotNetOpenAuth.Messaging.Bindings.ICryptoKeyStore CryptoKeyStore
	    {
	        get {
	            return  new InMemoryCryptoKeyStore();
	        }
	    }
	
	    public DotNetOpenAuth.Messaging.Bindings.INonceStore NonceStore
	    {
	        get { return new DummyNonceStore(); }
	    }
	
	    public AutomatedAuthorizationCheckResponse 
	             CheckAuthorizeClientCredentialsGrant(
	                         IAccessTokenRequest accessRequest)
	    {
	        throw new NotImplementedException();
	    }
	
	    public AutomatedUserAuthorizationCheckResponse   
	             CheckAuthorizeResourceOwnerCredentialGrant(
	                 string userName, string password, 
	                 IAccessTokenRequest accessRequest)
	    {
	        throw new NotImplementedException();
	    }
	}
```

*Listing 7*

```c#
	
	public static class Config
	{
	       public const string STS_CERT = "[…]";
	       public const string SERVICE_CERT = "[…]";
	}
```

## Generating and setting up certificates ##

Listing 8 includes commands which create the certificates necessary for testing and development scenarios. This is done by administrators in the Visual Studio prompt. In chronological order and with the makecert tool these commands create a CA_DNOA_DEMO root certificate, a revocation list for the root certificate and two certificates which are signed by the root certificate: DNOA_STS is created for signatures by the authorization server; DNOA_Service is designed for decoding the resource server. Subsequently, the generated certificates and the respective private keys are converted into pfx files which can be imported to the Windows Certificate Store.

To set up the certificates, the Management Console must be started (Start | Run | mmc) and the snap-in bearing the name Certificates is to be added for the local machine.

The root certificate is to be provided under trusted root certificates. The same procedure is applied to the revocation list with the ending crl. For this file to be selected, the given filter is to be changed to *.crl. In this way the pfx files (not the cer files) of the two remaining certificates are stored in the Personal folder. The filter must therefore also be modified.

Moreover, it must be ensured that IIS as well as the current user have access to the files where the Certificate Store stores the private certificates.

*Listing 8*

	makecert -n "CN=CA_DNOA_DEMO" -pe -r -sv ca_dnoa_demo.pvk ca_dnoa_demo.cer
	makecert -crl -n "CN=DA_DNOA_DEMO" -r -sv ca_dnoa_demo.pvk ca_dnoa.crl
	makecert -iv ca_dnoa_demo.pvk -n "CN=DNOA_STS" -ic ca_dnoa_demo.cer -sv dnoa_sts.pvk dnoa_sts.cer -sky exchange -pe -a sha1
	makecert -iv ca_dnoa_demo.pvk -n "CN=DNOA_Service" -ic ca_dnoa_demo.cer -sv dnoa_service.pvk dnoa_service.cer -sky exchange -pe -a sha1
	pvk2pfx -pvk dnoa_sts.pvk -spc dnoa_sts.cer -pfx dnoa_sts.pfx -po P@ssw0rd
	pvk2pfx -pvk dnoa_service.pvk -spc dnoa_service.cer -pfx dnoa_service.pfx -po P@ssw0rd

*Listing 9*

```c#
	
	public class CryptoKeyStoreEntry
	{
		public string Bucket { get; set; }
	    public string Handle { get; set; }
	    public CryptoKey Key { get; set; }
	}
```

*Listing 10*

```c#
	public class InMemoryCryptoKeyStore: ICryptoKeyStore {
	
		private static List keys = new List();
	
		[MethodImpl(MethodImplOptions.Synchronized)]
		public CryptoKey GetKey(string bucket, string handle)
		{
			return keys.Where(k => k.Bucket == bucket && k.Handle == handle)
									.Select(k => k.Key)
									.FirstOrDefault();
		}
	
		[MethodImpl(MethodImplOptions.Synchronized)]
		public IEnumerable<KeyValuePair> GetKeys(string bucket)
		{
			return keys.Where(k => k.Bucket == bucket)
								   .OrderByDescending(k => k.Key.ExpiresUtc)
								   .Select(k => new KeyValuePair(k.Handle, k.Key));
		}
	
		[MethodImpl(MethodImplOptions.Synchronized)]
		public void RemoveKey(string bucket, string handle)
		{
			keys.RemoveAll(k => k.Bucket == bucket && k.Handle == handle);
		}
	
		[MethodImpl(MethodImplOptions.Synchronized)]
		public void StoreKey(string bucket, string handle, CryptoKey key)
		{
			var entry = new CryptoKeyStoreEntry();
			entry.Bucket = bucket;
			entry.Handle = handle;
			entry.Key = key;
			keys.Add(entry);
		}
	}
```

*Listing 11*

```c#
	class DummyNonceStore : INonceStore
	{
	       public bool StoreNonce(string context, string nonce, DateTime timestampUtc)
	       {
	               return true;
	       }
	}
```

If the developer has completed implementation of the IAuthorizationServerHost interface, he can start implementing the MVC controller for the authorization server. An example of this is given in Listing 12. The OAuth2 flow starts with the Auth action method. The client refers the user to this method. Using an AuthorizationServer instance, to which the discussed implementation of IAuthorizationServerHost is passed, the client's OAuth2 request is identified. This is done using the ReadAuthorizationRequest method. The current request object describing the underlying HTTP request and which, among others, grants access to the URL parameters conforming to OAuth2 which were transferred by the client is transferred. The result of this method is an object describing the OAuth2 request. This object is stored in the current session to be used later on. Subsequently, the action method in question transfers to a view. This view aims at requesting the user to enter his username and password and asks for the confirmation of the client's authorization request.

If this has been completed, the overloading which has been annotated using HttpPost is initiated by Auth. It verifies username and password. To simplify this example, the user data storage is also hard-coded. A production-ready implementation would rather check against a database. To avoid timing attacks, DotNetOpenAuth offers the MessagingUtilities.EqualsConstantTime method which may be used instead of the comparison operator (==) when comparing passwords. Both of these pieces of information are to be found in the LoginModel (Listing 13) which has been passed. If they fit together, they are retrieved by the OAuth request which was previously stored in the session and a new AuthorizationServer instance is created, based on the developed IAuthorizationServerHost implementation. Consequently, the action method discussed approves the authorization request with PrepareApproveAuthorizationRequest. For this it transfers the OAuth2 request, the username and the Scope. Moreover, it transfers the result of this method into an ActionResult using AsActionResult, which is returned and subsequently makes ASP.NET MVC transfer the user to the web client. During this process, an authorization code is transferred, as discussed above, which the client may exchange for the desired token. For this to happen the Token action method is called, whereby the authorization code is expected as URL parameter by the AuthorizationServer instance. Similarly, the RejectAuthorizationRequest method could be used to reject the authorization request.

*Listing 12*

```c#
	
	public class OAuthController : Controller
	{
	       public ActionResult Auth()
	       {
	               var authSvr = new AuthorizationServer(new AuthServerHostImpl());
	               var request = authSvr.ReadAuthorizationRequest(Request);
	               Session["request"] = request;
	               return View();
	       }
	
	       [HttpPost]
	       public ActionResult Auth(LoginModel loginData)
	       {
	               var authSvrHostImpl = new AuthServerHostImpl();
	               var ok = (loginData.Username == "Max Muster" && loginData.Password == "test123");
	               if (ok)
	               {
	                      var request = Session["request"] as EndUserAuthorizationRequest;
	                      var authSvr = new AuthorizationServer(authSvrHostImpl);
	                      var approval = authSvr.PrepareApproveAuthorizationRequest(request, loginData.UserName, new[] { "http://localhost/demo" });
	
	                      return authSvr
	                               .Channel
	                               .PrepareResponse(approval)
	                               .AsActionResult();
	               }
	
	               ViewBag.Message = "Wrong username or password!";
	               return View();
	       }
	
	       public ActionResult Token()
	       {
	               var authSvr = new AuthorizationServer(new AuthServerHostImpl());
	               var response = authSvr.HandleTokenRequest(Request);
	               return response.AsActionResult();
	       }
	}
```

*Listing 13*

```c#

	public class LoginModel
	{
		public string Username { get; set; }
		public string Password { get; set; }
	}

```

## Accessing the authorization server and the resource server via the client ##

Implementing the client in order to have access to the user-defined authorization server is similar to implementing the client for the Google scenario (see Listing 1 and Listing 2). It goes without saying that the URLs would have to be changed and the identifier http://localhost/demo would have to be used as Scope. Listing 14 demonstrates the adapted implementation of the callback method OAuthCallback. Again, the ProcessUserAuthorization method is responsible for requesting the token by presenting the authorization code. Subsequently, the token is used for access to the resource server.

To show how this token can also be used for the authentication of the user, the discussed method asks for more information about the token using the token and then validates this information in accordance with the requirements of the current OpenId Connect [3] version and using a TokenValidator (Listing 7). The information about the token is obtained from the authorization server, here acting as the resource server.

The implementation of this action method – initiated with this call – is to be found in Listing 15. The action method creates an instance of the ResourceServer class which is based on a StandardAccessTokenAnalyzer. The latter knows the keys which may be used for checking the signature and decoding the token. By calling GetAccessToken, the action method reveals the decoded version of the transferred token. GetAccessToken receives the current ASP.NET request object and the expected Scope. If this scope is not to be found in the token, it throws an exception. The same happens when there are problems with checking the signature or decoding. Once the action method has received the token in this way, it returns information from the token. The audience property, saying for whom the token was issued, is important for validation by the client.

*Listing 14*

```c#

	private ActionResult OAuthCallback()
	{
		var auth = client.ProcessUserAuthorization(this.Request);
		var hc = new HttpClient();
		hc.DefaultRequestHeaders.Authorization = 
		         new AuthenticationHeaderValue("Bearer", auth.AccessToken);
		var tokenInfoUrl = "http://localhost/sts/oauth/TokenInfo";
		dynamic tokenInfo = 
		      hc.GetAsync(tokenInfoUrl).Result.Content.ReadAsAsync().Result;
		
		var tv = new TokenValidator();
		tv.ValidateToken(tokenInfo, expectedAudience: "RP");
		
		string info = tokenInfo.user + ", " + tokenInfo.audience;            
		
		return Content(info);
	}
```

*Listing 15*

```c#

	public ActionResult TokenInfo()
	{
	   var signCert = LoadCert(Config.STS_CERT);
	   var encryptCert = LoadCert(Config.SERVICE_CERT);
	
	   var analyzer = new StandardAccessTokenAnalyzer(
	                   (RSACryptoServiceProvider)signCert.PublicKey.Key,
	                       (RSACryptoServiceProvider)encryptCert.PrivateKey);
	
	   var resourceServer = new ResourceServer(analyzer);
	
	   var token = resourceServer.GetAccessToken(
	                      Request, new[] { "http://localhost/demo"});
	
	   return Json(
	       new { audience=token.ClientIdentifier, user=token.User }, 
	       JsonRequestBehavior.AllowGet);
	}
```

## OAuth2 for accessing services via traditional clients ##

Up to now, only web clients have been used in the examples given. However, OAuth2 can also be used to authorize a traditional client to access a service in the name of the user. If it is not possible for the user to inform the client about his password for this purpose, the client is forced to follow the flow described in the previous sections in a browser window which he hosts himself. In this case, the value urn:ietf:wg:oauth:2.0:oob is used as a callback URL, which makes the authorization server display the authorization code instead of the redirection in the browser window. There, the client can pick up the code to exchange it for the desired token as discussed above. The examples which are shown in the download version of DotNetOpenAuth include a WPF project demonstrating this variation.

Nevertheless, OAuth2 also supports scenarios in which the user tells the client his username and password. In such cases, the client transfers this information as well as the client ID and the client secret and the desired scope to the authorization server and then obtains the desired token directly without another call.

In order to enable this operation, the method for implementing IAuthorizationServerHost used by the authorization server (AutomatedUserAuthorizationCheckResponse) must be implemented (Listing 16). The task of this method is to check whether the mentioned user with the given password actually exists, and, if so, if he is allowed to delegate the requested right (Scope) to a client.

The result of type AutomatedUserAuthorizationCheckResponse contains the IAccessTokenRequest that DotNetOpenAuth has passed to the method, a Boolean indicating whether the user is allowed to delegate the requested right as well as a canonical username, which represents the official username of the given user. Differentiation can be made between this canonical username and the transferred username in terms of capitalization in particular.

*Listing 16*

```c#

	public AutomatedUserAuthorizationCheckResponse CheckAuthorizeResourceOwnerCredentialGrant(string userName, string password, IAccessTokenRequest accessRequest)
	{
	
	    if (userName != "Max Muster" || password != "test123")
	    {
	        return new AutomatedUserAuthorizationCheckResponse(
	                                    accessRequest: accessRequest, 
	                                    approved: false, 
	                                    canonicalUserName: null);
	    }
	
	    if (accessRequest.Scope.Count != 1 
	          || accessRequest.Scope.First() != "http://localhost/demo")
	    {
	        return new AutomatedUserAuthorizationCheckResponse(
	                                    accessRequest: accessRequest,
	                                    approved: false,
	                                    canonicalUserName: null);
	    }
	
	    return new AutomatedUserAuthorizationCheckResponse(
	                                    accessRequest: accessRequest,
	                                    approved: true,
	                                    canonicalUserName: userName);
	}
```

To obtain the desired token, the client uses the UserAgentClient class which is very similar to the WebClient class used by the web client (Listing 17). It is also based on an AuthorizationServerDescription which, among others, provides information about the endpoints of the authorization server. With ExchangeUserCredentialForToken the client can exchange the username and the password of the user for the desired token.

*Listing 17*

```c#

	private static IAuthorizationState GetAccessTokenFromOwnAuthSvr()
	{
	   var server = new AuthorizationServerDescription();
	   server.TokenEndpoint = new Uri("https://localhost/STS/OAuth/Token");
	   server.ProtocolVersion = DotNetOpenAuth.OAuth2.ProtocolVersion.V20;
	
	   var client = new UserAgentClient(server, clientIdentifier: "RP");
	
	   client.ClientCredentialApplicator = 
	                ClientCredentialApplicator.PostParameter("data!");
	
	   var token = client.ExchangeUserCredentialForToken(
	                 "Max Muster", "test123", new[] { "http://localhost/demo"});
	
	   return token;
	}
```

Once the client has obtained the token, he can use it to access services as usual (Listing 18). An example of such a service method which of course also has to check the transferred token is to be found in Listing 19. The OAuth2 attribute is used to verify the token, the implementation of which is described in Listing 20. This attribute acts as an authorization filter and takes over no, one, or more identifiers via the provided constructors which are expected in the scope of the transferred token. The transferred token is identified via the OnAuthorization method using the GetAccessToken method offered by the ResourceServer class and is decoded during this process. Moreover, the signature and the defined Scope are checked. If these actions fail, an exception is initiated and the processing of the request is cancelled. To avoid the need to perform these checks for every call, a session cookie or a respective session entry could be created if the action has been successful. This can be done easily using, for example, the membership provider of ASP.NET.

*Listing 18*

	var token = GetAccessTokenFromOwnAuthSvr();
	HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://localhost/RP/api/demo");
	request.Headers.Add("Authorization", "Bearer " + token.AccessToken);
	var response = request.GetResponse();
	var msg = new StreamReader(response.GetResponseStream()).ReadToEnd();
	Console.WriteLine(msg);

*Listing 19*

	[OAuth2("http://localhost/demo")]
	public string Demo()
	{
	       return "Hello World";
	}

*Listing 20*

	public class OAuth2Attribute: FilterAttribute, IAuthorizationFilter
	{
	   String[] scopes = new String[0];
	
	       public OAuth2Attribute() { }
	       public OAuth2Attribute(String scope) { scopes = new String[] { scope }; }
	       public OAuth2Attribute(String[] scopes) { this.scopes = scopes; }
	
	       public void OnAuthorization(AuthorizationContext filterContext)
	
	               var signCert = AuthHelper.LoadCert(Config.STS_CERT);
	               var encryptCert = AuthHelper.LoadCert(Config.SERVICE_CERT);
	
	               var analyzer = new StandardAccessTokenAnalyzer(
	                              (RSACryptoServiceProvider)signCert.PublicKey.Key,
	                              (RSACryptoServiceProvider)encryptCert.PrivateKey);
	
	               var resourceServer = new ResourceServer(analyzer);
	
	               var token = 
	                    resourceServer.GetAccessToken(
	                          filterContext.HttpContext.Request, scopes);
	       }
	}

## OAuth vs. OAuth2 ##

Although OAuth2 is nowadays already supported by large players such as Google or Facebook, and is the first choice for new developments, there are still platforms using the initial version of OAuth. DotNetOpenAuth also offers support for the latter. The respective object model is very similar to OAuth2's object model, but, understandably, it tends to use the vocabulary used by the initial OAuth specification. Examples of this are given in the download package of DotNetOpenAuth.
Summary

DotNetOpenAuth supports the implementation of OAuth2 scenarios. Developers need not deal with the implementation of the protocol described therein. However, developers are advised to become acquainted with the specification of OAuth2 (http://tools.ietf.org/html/draft-ietf-oauth-v2-30) to get used to the vocabulary used in this sector and to understand design decisions which DotNetOpenAuth is based on.

Moreover, developers should understand that OAuth2 is mainly used to authorize clients, although it is increasingly used for authentication and single sign-on. The security loopholes created thereby are closed by the emerging OpenId Connect standard and providers such as Google or Facebook already recommend measures correlating with it. Therefore, it also seems to be useful to deal with OpenId Connect although this standard is still being developed.