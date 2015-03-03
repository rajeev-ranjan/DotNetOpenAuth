You have two options when it comes to logging and debugging DotNetOpenAuth. Both options utilise the very popular log4net logging framework.

but first,

> As a preamble, be advised that logging can potentially emit messages that could compromise the security of your system if those logs were leaked.  Specifically log messages may include personally identifiable information or even symmetric secrets that might enable user identity spoofing.  
> Log responsibly: store your logs in a secure place.

### Requirements

Make sure to use log4net version 1.2.11 exactly.

## Option 1 - Logging to Glimpse

Glimpse is an excellent utility which provides a developer with  information on what is going on underneath the hood of his webserver, and presents the information in a similar way to Firebug (a very popular web development plugin for Firefox).

A plugin for Glimpse has been written that exposes the internal logging within DotNetOpenAuth to the Glimpse framework, and installation is VERY simple.

To install the plugin simply add a package reference to your web project via the Visual Studio UI or directly from the Package Console using the command

```
Install-Package DCCreative.DNOA4Glimpse
```

This will automatically add the necessary references to your project and configure logging for you.

Once this has completed you need to activate Glimpse by visiting http://yourwebsiteurl.example.com/glimpse.axd and switching Glimpse on. This should result in a panel being displayed at the bottom of your screen, and if everything has gone to plan you should see a most awesome DotNetOpenAuth tab.

Further details can be found on this post by David Christiansen.

## Option 2 - Logging directly with Log4Net

1. Make sure you have log4net.dll referenced in your project, or if you are developing a asp.net website (not website project) make sure log4net.dll is in your web site's Bin directory.

2. Activate Log4net logging in an override of the Application Start event For example in Global.asax.cs (or alike):

```c#
private void Application_Start(object sender, EventArgs e) {
	log4net.Config.XmlConfigurator.Configure();
}
```

3. Then update your web.config file to include a section definition for log4net in configSections AND lastly the actual section definition. The following config example will log ALL events raised by DotNetOpenAuth to a text log file;

```xml
<configuration>
	<configSections>
		<section name="log4net" type="log4net.Config.Log4NetConfigurationSectionHandler, log4net" requirePermission="false" />
	</configSections>

	<!-- log4net is a 3rd party (free) logger library that dotnetopenid will use if present but does not require. -->
	<log4net>
		<appender name="RollingFileAppender" type="log4net.Appender.RollingFileAppender">
			<file value="RelyingParty.log" />
			<appendToFile value="true" />
			<rollingStyle value="Size" />
			<maxSizeRollBackups value="10" />
			<maximumFileSize value="100KB" />
			<staticLogFileName value="true" />
			<layout type="log4net.Layout.PatternLayout">
				<conversionPattern value="%date (GMT%date{%z}) [%thread] %-5level %logger - %message%newline" />
			</layout>
		</appender>
		<!-- Setup the root category, add the appenders and set the default level -->
		<root>
			<level value="INFO" />
			<appender-ref ref="RollingFileAppender" />
		</root>
		<!-- Specify the level for some specific categories -->
		<logger name="DotNetOpenAuth">
			<level value="ALL" />
		</logger>
	</log4net>
</configuration>
```

### Additional Sources of Information

You can find alternative configurations for Log4Net on the [Official Log4Net Config Examples page](http://logging.apache.org/log4net/release/config-examples.html)
