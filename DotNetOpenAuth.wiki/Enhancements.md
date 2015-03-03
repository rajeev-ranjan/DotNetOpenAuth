# Behaviours

DotNetOpenAuth offers "behaviors", which automatically modify outgoing or incoming OpenID messages in a standard way. One built-in behavior is the AXFetchAsSregTransform behavior, which for the relying party automatically translates requests that include the Simple Registration extension into a request that includes that extension and/or any of three formats of Attribute Exchange extensions depending on the Provider you're authenticating the user with. By activating this behavior in your site, all you have to do is use ClaimsRequest and ClaimsRespones, and you'll have a very high chance of getting the attributes you're requesting if the Provider supports attributes at all.
Activating AXFetchAsSregTransform

Whether you're using the ASP.NET controls or using the 'programmatic' approach, activate the behavior in your Web.config file. Below is a minimal file that should be merged with your own:

```xml
<configuration>
   <configSections>
      <sectionGroup name="dotNetOpenAuth" type="DotNetOpenAuth.Configuration.DotNetOpenAuthSection, DotNetOpenAuth.Core">
          <section name="openid" type="DotNetOpenAuth.Configuration.OpenIdElement, DotNetOpenAuth.OpenId" requirePermission="false" allowLocation="true" />
      </sectionGroup>
   </configSections>
   <dotNetOpenAuth>
      <openid>
         <relyingParty>
            <behaviors>
               <!-- The following OPTIONAL behavior allows RPs to use SREG only, but be compatible
                    with OPs that use Attribute Exchange (in various formats). -->
               <add type="DotNetOpenAuth.OpenId.Behaviors.AXFetchAsSregTransform, DotNetOpenAuth" />
            </behaviors>
         </relyingParty>
      </openid>
   </dotNetOpenAuth>
</configuration>
```

## ASP.NET Controls users

The ASP.NET controls do the right thing. Just set the properties on the controls to request the attributes you want and it will Just Work.

## Programmatic notes

With the behavior activated, only add ClaimsRequest (sreg) to your IAuthenticationRequest instance -- don't add a FetchRequest (AX) extension. When you redirect the user to the Provider, the behavior will auto-detect the Provider's capabilities of accepting sreg and AX, and modify the outgoing request accordingly. When the response comes back, the behavior will inspect the message and translate any AX response back into sreg for you.

## Special Provider notes

No Provider offers all attributes for all users. Most large Providers only offer a few attributes on their users like email and possibly timezone. Request any and all attributes you want, and be prepared to deal with whatever subset of attributes you get back.

Google has one unique trait, in that it ignores all attribute requests marked as 'optional'. You must request the user's email address as 'required' in order to ever get an email address from Google. Be wary though, that by marking the attribute as required, Google will refuse to authenticate the user unless the user is willing to give up their email address. So if you don't actually require the email address, it may be best to mark it as optional, and just forego getting it from your Google users in order to avoid chasing your users away by forcing them to give up their email address if they don't want to.
