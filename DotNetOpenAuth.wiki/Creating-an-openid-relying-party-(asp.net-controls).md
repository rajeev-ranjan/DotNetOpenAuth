All ASP.NET pages that use drop-in OpenID controls must include this line near the top of the page:

```c#
<%@ Register Assembly="DotNetOpenAuth" Namespace="DotNetOpenAuth.OpenId.RelyingParty" TagPrefix="rp" %>
```

Complete login form that automatically hooks into FormsAuthentication

```c#
<rp:OpenIdLogin ID="OpenIdLogin1" runat="server" />
```

**Pros:** Very easy, complete OpenID login solution, with some customization capability.
**Cons:** Uses HTML tables for layout. Not that flexible if it doesn't meet your needs.

**Gotchas:** The control automatically calls FormsAuthentication.RedirectFromLoginPage after firing the LoggedIn event unless that event handler sets e.Cancel = true.
Simple OpenID textbox for more control over UI but minimal code

In your ASPX page add this tag where you want the text box to appear:

```c#
<rp:OpenIdTextBox ID="OpenIdTextBox1" runat="server" />
```

You supply your own login button, and its event handler should include this code:

```c#
OpenIdTextBox1.LogOn();
```

**Pros:** Pretty easy... just one line of code.
**Cons:** none
**Gotchas:** The control automatically calls FormsAuthentication.RedirectFromLoginPage after firing the LoggedIn event unless that event handler sets e.Cancel = true.
OpenID Ajax login control

For blog comments and other logins where the page doesn't need to change after a login is completed, this control may be appropriate:

```c#
<rp:OpenIdAjaxTextBox ID="OpenIdAjaxTextBox1" runat="server" />
```

No login button is necessary, as it begins logging the user in immediate after it loses input focus. You can optionally add extensions to retrieve information about the user from the Provider by hooking the control's LoggingIn event:

```c#
protected void OpenIdAjaxTextBox1_LoggingIn(object sender, OpenIdEventArgs e) {
        // Retrieve the email address of the user
        e.Request.AddExtension(new ClaimsRequest {
                Email = DemandLevel.Request,
        });
}
```

Since this is AJAX, you probably want access to this email address in Javascript on the web page, so you hook the UnconfirmedPositiveAssertion event in the page code-behind:

```c#
protected void OpenIdAjaxTextBox1_UnconfirmedPositiveAssertion(object sender, OpenIdEventArgs e) {
        // This is where we register extensions that we want to have available in javascript
        // on the browser.
        OpenIdAjaxTextBox1.RegisterClientScriptExtension<ClaimsResponse>("sreg");
}
```

And then you hook the client-side authentication event in order to process the email address when it comes in:

```javascript
<script type="text/javascript">
	function onauthenticated(sender) {
		var emailBox = document.getElementById('<%= emailAddressBox.ClientID %>');
		emailBox.disabled = false;
		emailBox.title = null; // remove tooltip describing why the box was disabled.
		// the sreg response may not always be included.
		if (sender.sreg) {
			// and the email field may not always be included in the sreg response.
			if (sender.sreg.email) { emailBox.value = sender.sreg.email; }
		}
	}
</script>
```

When the page finally has a postback and you want to find out for sure who logged in (in Javascript nothing can be proven... only assumed), you can handle the LoggedIn event:

```c#
protected void OpenIdAjaxTextBox1_LoggedIn(object sender, OpenIdEventArgs e) {
	string claimedId = e.Response.ClaimedIdentifier;
	// Do something here
}
```

Having wired up all these events, the tag for the control itself looks something like:

```aspnet
<openid:OpenIdAjaxTextBox ID="OpenIdAjaxTextBox1" runat="server" CssClass="openidtextbox"
        OnLoggingIn="OpenIdAjaxTextBox1_LoggingIn"
        OnLoggedIn="OpenIdAjaxTextBox1_LoggedIn"
        OnClientAssertionReceived="onauthenticated(sender)"
        OnUnconfirmedPositiveAssertion="OpenIdAjaxTextBox1_UnconfirmedPositiveAssertion" />
```

Pros: Slick AJAX behavior makes it very easy for users familiar with OpenID to login.

Cons: Like the other controls, it works best in ASP.NET web forms rather than MVC.