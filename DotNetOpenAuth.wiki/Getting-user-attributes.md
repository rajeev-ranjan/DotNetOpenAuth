There are two ways to send a Simple Registration request for attributes from your OpenID relying party site to the OpenID Provider in order to obtain user attributes during authentication.

Note that obtaining user attributes without actually authenticating the user (effectively: tell me about the user without telling me who the user is) is supported as well in this library per the OpenID spec, but it's not supported by any public Provider out there that I know of.

#### ASP.NET controls

The OpenIdTextBox and OpenIdLogin ASP.NET controls that ship with DotNetOpenAuth include properties that automatically add a Simple Registration extension to the authentication request.

Here is a snippet of using the OpenIdTextBox control in such a way that during authentication, the user's email address will be "required" of the Provider while the user's postal code will be "requested" of the Provider.

```
<rp:OpenIdTextBox runat="server" ID="OpenIdTextBox1"
    OnLoggedIn="OpenIdTextBox1_LoggedIn"
    RequestEmail="Require"
    RequestPostalCode="Request" />
```

Several more attributes can be requested using Simple Registration. Look for the properties on this and the OpenIdLogin control that start with "Request".

When authentication returns successfully, these attribute values MAY (the Provider does not have to provide the values) be retrieved with this code:

```c#
protected void OpenIdTextBox1_LoggedIn(object sender, OpenIdEventArgs e) {
   var sreg = e.Response.GetExtension<ClaimsResponse>();
   if (sreg != null) {
      var email = sreg.Email;
      var postalCode = sreg.PostalCode;
      // Do something with the values here, like store them in your database for this user.
   }
}
```
