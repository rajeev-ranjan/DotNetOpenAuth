# Contributing?

That's fantastic. This library relies upon the contributions of time, effort, resources and finance from individuals and companies. Significant investment in such areas has got us to where we are today, but we always need more - Without help from people such as yourself, there would not be a DotNetOpenAuth library.

## Starting out

First, check out the Developers Quickstart, then the coding guidelines.

## GIT Source Control

We use GitHub to host the DotNetOpenAuth project. If you are new to GIT, we recommend you give this series of articles a read.

GitHub makes it very easy to fork projects so that you can make your own changes to a project and optionally get your changes incorporated back into the original project. If you want the source code so you can make changes to it, whether or not you are interested in contributing your changes back here, forking may be a good way to track what changes you make, and allow you convenient sharing of patches between the two projects.

## Branch management

Whether you contribute directly to the main project or to a fork of the project, it is important to understand how branches are managed in this project. The **master** branch is where most feature work occurs. It is generally not a stable branch and should not be used in production. As the next major release approaches, we branch off of master with a name of the convention vx.y. At this point, only bug fixes get checked into this stabilization branch. Once the branch is stable enough for a release, we tag a commit with the full version number of that release.

A single vx.y branch may provide several releases. For example, the v2.5 branch has been tagged at versions v2.5.0.8281, v2.5.1.8313, etc. when sufficient bug fixes since the last release in a particular branch justifies another minor release.

Bug fixes should be checked into the earliest branch to which it applies. For example, if a bug is discovered that affects the v3.1, v3.2, and master branches, we check the bug fix into v3.1 only. Before a release (and periodically in between releases) we merge from earlier branches to later branches in order to roll up bug fixes to all later versions. This means that some bug fixes will only exist in v3.1 for several weeks, leaving the bug present in v3.2 and later in the meantime. This turns out to usually not be a big problem. If it is, we can just merge the v3.1 branch into the v3.2 branch, or do a git cherry-pick to get the bug fix into the other branch faster.

## Getting the code

### Command-Line Access

If you plan to make changes either for yourself or to contribute back, use this command to check out the code using Git:

```
git clone git://github.com/AArnott/dotnetopenid.git
```

### Browser Access

If you only want a source code drop and do not plan on contributing changes back, you may also use your browser to visit the following URL:

**Repo Url:** git://github.com/DotNetOpenAuth/DotNetOpenAuth.git

From here you may download individual files or "snapshots" of anything you like. If you download a snapshot and you're using Windows, you'll need to extract the .tar.gz file. You can use WinZip for this, or read up on how to extract source from .tar.gz files using FOSS (but the FOSS-way is harder).
Getting Git

Download Git: [Windows](http://code.google.com/p/msysgit/).

[Crash Course on Git for SVN users](http://git.or.cz/course/svn.html)

### Which branch should you use?

On the web page for our repository, at the bottom you'll see a section called "heads". These are our branches. "master" is where most of the active work goes, while the others are our stabilization branches that eventually turn into released versions. When a version is released an entry under the "tags" section is added.

If you're planning on using this library in a production web site, avoid using master, as we cannot guarantee its readiness for production web sites. For production use, you should download a tagged release, or at very least go for one of the stabilization branches.
Commit access

### Code ownership

Notwithstanding this is an open-source project, there is a sense of ownership in the code contributors submit via check-in or patches. Ownership can explicitly transfer when the owner asks or authorizes someone else to take over a feature. Ownership can implicitly transfer if a contributor has not participated by email or coding contributions in the last several months and someone else picks up ownership, usually by studying the code and fixing bugs in it.

Coding etiquette suggests that before you change code you didn't write, you should email the owner of the code and clear it with him or her first, perhaps with the proposed patch.

### Coding Style

    Tabs, not spaces. And curly-braces usually not on their own lines. See quick start for how to configure VS' auto-formatting to fit these rules.
    Comply with all StyleCop rules, including the ones added in quick start.

### Committing changes

Generally speaking, commits are bug fixes or feature work, and there should be tickets open for the work you're doing and assigned to you before you begin work so others know what you're working on and don't do work in the same area.

All commits should be carefully scoped. Resist the urge to change anything not directly related to the task you're working on. If you see another change you'd like to make, make a separate commit for that change. Some commits might be "Fixed a bunch of FxCop messages" and that's fine. But it's not fine to fix some FxCop messages in foo.cs while implementing a feature in bar.cs and commit them both together.

Before git commit, review these:

    **ALWAYS **review the diff of your change before committing it and make sure that there is no diff noise from frivolous whitespace changes or irrelevant code churn.
    **DO **write unit tests to verify a fix or feature and include in the same or the next commit.
    **DO **make many commits as necessary to isolate changes. For instance, if you are adding a feature and find a bug in the process, fix the bug and check that fix in separately, then resume work on your feature.
    **DO **break a large feature into several smaller commits if it is architecturally sound to do so and each commit adds value.
    **DON'T **fix multiple bugs in one commit if you can avoid it.
    **DON'T **break the build. Every commit should build and (preferably) pass unit tests.
    **DO **verify that any .js changes you made in the library minify in release builds without introducing errors (do not omit required semicolons).

Before git push, make sure you satisfy all these criteria:

    StyleCop should run clean (Ctrl+Shift+Y in the IDE).
    All unit tests should pass.
    FxCop should run clean.

Remember that once you push, you must not rewrite history. Any corrections you need to make must be an additional commit.

After a git push, be sure to resolve any tickets your work has completed.

### Goals

As many developers from various OS and IDE backgrounds contribute to this project, we seek to achieve a uniform coding style for these reasons:

* Stay productive by focusing on code content and not perpetually debating or changing coding style.
* Easy reading and maybe fewer bugs if we're lucky.
* Predictable conventions based on generally accepted industry practice for C# programs and libraries.
* Leverage the power of .NET wherever possible to reduce code size and testing requirements.
* Allows for clean FxCop (code analysis) runs with an agreed upon set of rules.
* Minimize whitespace code churn. Diffs between versions should cleanly represent changes to code and not spurious changes due only to whitespace formatting changes made by an IDE or an errant programmer.

###Strict rules

Exception messages should never include confidential information (association secrets, usernames, etc.) as these can and do get sent to untrusted remote parties in some cases.