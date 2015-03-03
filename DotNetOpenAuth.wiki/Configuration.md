The following is a configuration file that includes ALL the settings that DotNetOpenAuth supports. All of them are optional, and are included here with their default values.

```xml
<configuration>
    <configSections>
        <section name="uri" type="System.Configuration.UriSection,
            System, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089" />
        <section name="dotNetOpenAuth" type="DotNetOpenAuth.Configuration.DotNetOpenAuthSection"
            requirePermission="false" allowLocation="true"/>
    </configSections>
 
    <!-- The uri section is necessary to turn on .NET 3.5 support for IDN (international domain names),
         which is necessary for OpenID urls with unicode characters in the domain/host name.
         It is also required to put the Uri class into RFC 3986 escaping mode, which OpenID and OAuth require. -->
    <uri>
        <idn enabled="All"/>
        <iriParsing enabled="true"/>
    </uri>
 
    <system.net>
        <defaultProxy enabled="true" />
        <settings>
            <!-- This setting causes .NET to check certificate revocation lists (CRL)
                 before trusting HTTPS certificates.  But this setting tends to not
                 be allowed in shared hosting environments. -->
            <servicePointManager checkCertificateRevocationList="true"/>
        </settings>
    </system.net>
 
    <dotNetOpenAuth>
        <openid maxAuthenticationTime="0:05" cacheDiscovery="true">
            <relyingParty>
                <security
                    requireSsl="false"
                    minimumRequiredOpenIdVersion="V10"
                    minimumHashBitLength="160"
                    maximumHashBitLength="256"
                    requireDirectedIdentity="false"
                    requireAssociation="false"
                    rejectUnsolicitedAssertions="false"
                    rejectDelegatingIdentifiers="false"
                    ignoreUnsignedExtensions="false"
                    protectDownlevelReplayAttacks="true"
                    privateSecretMaximumAge="07:00:00" />
                <behaviors>
                    <!-- <add type="Fully.Qualified.ClassName, Assembly" /> -->
                </behaviors>
                <store type="Fully.Qualified.ClassName, Assembly" />
            </relyingParty>
            <provider>
                <security
                    requireSsl="false"
                    protectDownlevelReplayAttacks="true"
                    unsolicitedAssertionVerification="RequireSuccess|LogWarningOnFailure|NeverVerify"
                    minimumHashBitLength="160"
                    maximumHashBitLength="512">
                    <associations>
                        <add type="HMAC-SHA1" lifetime="14.00:00:00" />
                        <add type="HMAC-SHA256" lifetime="14.00:00:00" />
                    </associations>
                </security>
                <behaviors>
                    <!-- <add type="Fully.Qualified.ClassName, Assembly" /> -->
                </behaviors>
                <store type="Fully.Qualified.ClassName, Assembly" />
            </provider>
            <extensionFactories>
                <add type="FullyQualifiedClass.Implementing.IOpenIdExtensionFactory, Assembly" />
            </extensionFactories>
            <xriResolver enabled="true" proxy="xri.net" />
        </openid>
        <messaging clockSkew="00:10:00" lifetime="00:03:00" strict="true">
            <untrustedWebRequest
                timeout="00:00:10"
                readWriteTimeout="00:00:01.500"
                maximumBytesToRead="1048576"
                maximumRedirections="10">
                <whitelistHosts>
                    <!-- since this is a sample, and will often be used with localhost -->
                    <!-- <add name="localhost" /> -->
                </whitelistHosts>
                <whitelistHostsRegex>
                    <!-- since this is a sample, and will often be used with localhost -->
                    <!-- <add name="\.owndomain\.com$" /> -->
                </whitelistHostsRegex>
                <blacklistHosts>
                </blacklistHosts>
                <blacklistHostsRegex>
                </blacklistHostsRegex>
            </untrustedWebRequest>
        </messaging>
    </dotNetOpenAuth>
</configuration>
```