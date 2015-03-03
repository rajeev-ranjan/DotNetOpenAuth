## Basic authentication

Relying Parties can be written using the ASP.NET web controls (easier) or programmatically (offering greater UI and functional flexibility).

### Getting user attributes

OpenID extensions offer easier user registration on your web site by allowing relying parties to auto-fill registration forms (perhaps skipping them altogether) by obtaining values for these fields directly from the user's OpenID Provider.

Two popular OpenID extensions exist for obtaining user attributes: Simple Registration and Attribute Exchange. You may be logging users in from Providers that support either one or both of these. A relying party such as yours should be prepared to interoperate using both of these extensions in order to provide the best login experience for all of your users.

To simplify interoperability between your relying party and all the Providers out there and which extensions they support for attributes, turn on the AXFetchAsSregTransform behavior to maximize your chance of getting attributes back from the Provider and then just use Simple Registration on your side while the translation of different extensions happens magically for you in the background.

## Boosting security

If you are running a relying party that has particularly high security requirements and the standard OpenID security profile isn't sufficient, there are things you can do to further boost security.

[Roadmap](wiki/Roadmap)