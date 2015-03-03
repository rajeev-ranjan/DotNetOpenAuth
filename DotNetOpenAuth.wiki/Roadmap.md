# v4.4

* Removal of dependency upon log4net.  Using [LibLog](https://github.com/damianh/LibLog) developers can easily switch out their own logging framework.

# v5.0

v5.0 is finishing up. Here are the highlights:
* API breaking changes allowed.
* Greatly simplified OAuth 1 consumer API
* Switch to C# 5 Async style methods to increase scalability of high traffic servers.
* Switch all HTTP traffic from DNOA to use HttpClient instead of HttpWebRequest, and use HttpRequestMessage instead of HttpRequestBase as the common element of describing an incoming request.
* Removal of all dependencies on Microsoft Code Contracts.
* More unit testable (by virtue of `HttpClient` use, full programmatic configurability, etc.)

# v5.1

Possible directions for the next release.
* Portable class library, targeting .NET 4.5, Windows Phone 8 and Windows Store apps ( **at risk** )
* Possible use of MEF for composing extensions, etc.
* Removal of all web.config options in favor of only programmatic configuration. (May be required by target platform not including config section types.)
