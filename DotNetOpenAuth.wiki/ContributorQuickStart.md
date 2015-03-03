This page contains instructions for how to configure your development environment to best work with DotNetOpenAuth so you can author high quality code that conforms to the coding guidelines this library adheres to.

### Prerequisites

DotNetOpenAuth requires several software packages to build that it does not require at runtime for its users. First make sure you have installed this software.

* [Visual Studio 2008 SP1](http://www.microsoft.com/downloads/details.aspx?FamilyId=FBEE1648-7106-44A7-9649-6D9F6D58056E&displaylang=en) - [Team Suite](http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=d95598d7-aa6e-4f24-82e3-81570c5384cb) required for building everything in the v3.3 branch and older, but lesser SKUs can build the core library and samples
* [Visual Studio 2010 Ultimate](http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=457bab91-5eb2-4b36-b0f4-d6f34683c62a) required for building everything in the v3.4 branch and later, but lesser SKUs can build the core library and samples
* [Microsoft StyleCop](http://code.msdn.microsoft.com/sourceanalysis/Release/ProjectReleases.aspx)
* [Microsoft DevLabs Code Contracts](http://msdn.microsoft.com/en-us/devlabs/dd491992.aspx)
* [Microsoft ASP.NET MVC](http://www.asp.net/mvc/download/)
* Git
* [Microsoft Web Deployment Tool](http://www.iis.net/extensions/WebDeploymentTool) (may be required to build the DotNetOpenAuth.BuildTasks project)
* [ILMerge](http://research.microsoft.com/en-us/people/mbarnett/ILMerge.aspx) - A utility for merging multiple .NET assemblies into a single .NET assembly ( Download)
* [Visual Studio Team System 2008 Database Edition GDR R2](http://www.microsoft.com/downloads/details.aspx?FamilyID=bb3ad767-5f69-4db9-b1c9-8f55759846ed&displaylang=en) - only required for building the project template for v3.3.  v3.4 and later branches only need Visual Studio 2010 as mentioned above.
* [Windows Installer XML (WIX)](http://sourceforge.net/projects/wix/files/)

In addition to installing the above software, you must configure it with these additional steps. If you're concerned about affecting other projects you work on, you can export your VS settings before and after these steps and swap them around whenever you work on DotNetOpenAuth. You can also turn off the new [StyleCop rules](http://blog.nerdbank.net/2008/09/notrailingwhitespace-stylecop-rule-and.html) for your other projects using the standard StyleCop UI.

    Copy lib\NerdBank.StyleCop.Rules.dll from the source code to your %PROGRAMFILES%\Microsoft StyleCop 4.3 directory. This will add some StyleCop rules to every StyleCop run that help your code to adhere to DotNetOpenAuth coding guidelines.
    Import the src\C# formatting rules.reg file into your registry. This sets the rules for where to place newlines when VS auto-formats your code.
    In VS, Tools -> Options -> Text Editor -> C# -> Tabs: Keep tabs.

Building DotNetOpenAuth

Because of bugs or design limitations of some of the build tools we use, we must always delay sign the build rather than either not signing or fully signing depending on what kind of a build we want. It's not ideal, but it makes it work. But that does mean a bit more upfront work for you to get it to build on your dev box.

To get the build to work in the IDE without doing any signing changes, you need to register for skip verification on a particular public key so that delay-signing works. Run this command at an elevated Visual Studio command prompt, and restart Visual Studio.

  sn -Vr *,2780ccd10d57b246

If you want to build the library for use on another computer, when compiling v3.3 of later, you need to generate your own signing key file and build with that. Here is how you can do that:

```
sn -k mykeyfile.pfx
sn -i mykeyfile.pfx mykeycontainer
sn -p mykeyfile.pfx mykeyfile.pub
sn -q -t mykeyfile.pub # to print the public key token that you just generated to the console
sn -Vr *,<YourPublicKeyTokenHere>
```

# any time you build at the command line

```
msbuild /p:KeyPairContainer=mykeycontainer,PublicKeyFile="<full path to your public key>mykeyfile.pub"
```

That's it. You're ready to start coding.