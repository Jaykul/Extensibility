---
theme: "night"
highlightTheme: "github-dark"
transition: "slide"
slideNumber: true
defaultTiming: 100
---

<!-- .slide: data-background-position="top center" data-background="./images/header.png" data-background-opacity="1" data-background-size="368px"  -->

# Extensibility  <!-- .element: class="r-fit-text" -->
## Patterns & Practices
#### Modules That are Just the Beginning
https://github.com/Jaykul/Extensibility

Joel "Jaykul" Bennett


<img src="./images/cc-by-nc-sa.png" align="right" width="200px" style="border: 0px">

note: Welcome to "Extensibility: Patterns and practices for modules that are just the beginning." I am, of course, Joel Bennett.
You may know me as "Jaykul" on the PowerShell Discord, Slack, or IRC, Mastodon, Twitter, or some other social media. I've been running the online virtual user group / chat community for about 16 years, and I am a fourteen-time PowerShell MVP.

---

<!-- .slide: data-background-position="top center" data-background="./images/header.png" data-background-opacity="1" data-background-size="368px"  -->

# About Me

Joel "Jaykul" Bennett

Principal DevOps Engineer

### Hacker

HuddledMasses.org

note: Before we get started today I want to tell you a little something about myself: Occupationally I'm a programmer. My business cards (if I had any) would say "Principal DevOps Engineer" or in the past "Software Engineer," but never "Software Architect" -- at the end of the day, I'm a hacker.

---

## I want to write modules that are:

1. **Usable** by _other people_
2. **Maintanable** by _me_
3. **Extensible** by _contributors_

note: It's important (to me) that you understand that, as we get into our subject for today. We're going to be talking a lot about design, but I want you to keep in mind _the reason_ we are putting this much thought into design. I don't have an multi-tier architecture in mind, and we're not trying to get to 3rd normal form, I'm designing to make things _more maintainable_, and _more extensible_.


---

<!-- .slide: data-background-position="top 0px left 0px" data-background="./images/bg.png" data-background-opacity="0.3"  data-background-size="228px"  -->

# Extension Points  <!-- .element: class="r-fit-text" -->

## Extension Modules <!-- .element: class="fragment" data-fragment-index="1" -->
## Hook Scripts <!-- .element: class="fragment" data-fragment-index="2" -->
## Base Commands <!-- .element: class="fragment" data-fragment-index="3" -->

note: I have three scenarios for extensibility that I've had to use in the last year or two.

note: The first is a **module** that allows extension by other modules.
note: The second is a module with **commands** that support extending by allowing users to provide scripts to be invoked at specific points.
note: The third is a module with a **base command** that can be called by other modules which provide content.

---

# Extension Modules  <!-- .element: class="r-fit-text" -->

Module relies on _other modules_ for implementation. Extension modules implement some interface, like a set of commands with specific parameters.

note: These are the easy ones for most people to understand. Basically something like SecretManagement, where other modules provide specific implementations of the functionality. Since we have the example of SecretManagement from the PowerShell team, most of us will be familiar with the idea. Of course, not all such modules have to be done that way. The base module might use one extension module at a time. It might call all of them in sequence until some result, or all at once in parallel. The key here is that there has to be some way to _discover_ the source modules. Let's talk about two models for this.

--

## Secret Management


### Discoverability:

```PowerShell
NestedModules = [ModuleName].Extension
```


### Interface:

```PowerShell
Get-Secret
Set-Secret
Remove-Secret
Get-SecretInfo
Test-SecretVault
```


note:
You've all seen the SecretManagement module, right? It can't store secrets on it's own. There are _other_ modules that implement actually storing the secrets.
:
The SecretManagement module discovers modules by looking for modules that have a nested module (usually in a subfolder) named "Module.Extension," and then loading the nested module.
:
Secret Vault modules also have to implement the "interface" of the Secret Management module: `Get-Secret`, `Set-Secret`, `Remove-Secret`, `Get-SecretInfo`, `Test-SecretVault`.

--

## EzTheme

### Discoverability:

```PowerShell <!-- .element: data-line-numbers="2-5"  -->
PrivateData = @{
  EzTheme = @{
    "Get" = ...
    "Set" = ...
  }
}
```

### Interface:

The commands can be named anything, can output anything, as long as this is valid.

```PowerShell
Get-MyConfig | Set-MyConfig
```

note: I think it's obvious that EzTheme is much easier to extend.
You don't have to make a special extra module, or implement specific command names.
Additionally, your command won't be invoked in a hidden PowerShell instance ;)
Importantly, your commands are usable by your users.

---

# Hook Scripts <!-- .element: class="r-fit-text" -->

Module has a complete, albeit basic, implementation. Hook scripts extend certain points of the work flow, adding functionality or validation.

note:
Think about git hooks, for example. Pre-commit, prepare-commit-msg, commit-msg, post-commit... pre-rebase, post-merge, pre-push, post-checkout, post-rewrite, etc. Git isn't incomplete without them, but they can be used to extend the functionality.

--

## Bicep Flex

A module I wrote for loanDepot. Creates resource group, enforces naming & tagging conventions, deploys bicep templates.

Supports extension of naming conventions, authentication/login, parameter generation, and post-deployment validation.

note: I haven't been able to release this publically yet, but it's relatively straightforward. You run it targetting a specific deployment, and it will run all the bicep files in a folder, and process hook scripts from that folder _or parent folders_, so we can group (for example) all our static website (CDN) deployments together, and have a set of hook scripts that handle (external) DNS registration and data plane operations like turning on the static site, https, etc.

---

# Base Commands <!-- .element: class="r-fit-text" -->

Module provides a complex base command. <br/>
Extensions are _other commands_ that call it and pass it content. They might expose all of its parameters.

note: It could be like `Show` in the `Show-UI` module (which can display any WPF controls), or the `Block` command in TerminalBlocks that can wrap any text. The idea in both of these cases was to make it easy for people to put their own content into our display/formatter.

note: One way to implement that is to just have **base commands** and hope that others will write commands that call it. Another way is what I did in the build process for TerminalBlocks, contribute extensions of those command. Each base command probably has a set of "common" parameters that extension functions need to pass through to the base command.

note: Our example here The extensions  Author wants contributions, rather than extensiondoes something other modules can call to provide content.


note:
Of course, the SecretManagement module requires you to module calls the commands in those modules, and those modules implement the commands. The Secret Management module doesn't care how the commands are implemented, it just calls them.

--

## Show-UI

### Base Command:

<image src="./images/ShowUIHelp.png">

note: Some of you may remember one of my older modules...
`Show-Window` (AKA Show-UI) was actually a pretty complex base command, since it basically supported everything that a WPF window could do.
And yet we wanted it to be _built in_ to every single Show-UI command that we generated, as a `-Show` parameter...
We actually did that with `Start-WpfJob` too, adding it as the `-AsJob` parameter to every command we generated.

--

## TerminalBlocks

### Base Command:

```text
New-TerminalBlock [[-Content] <Object>] [-Caps <BlockCaps>]
  [-Prefix <String>] [-Postfix <String>] [-Separator <String>]
  [-Position <TerminalPosition>] [-Alignment {Left | Right}]
  [-ForegroundColor <RgbColor>] [-BackgroundColor <RgbColor>]
  [-AdminFg <RgbColor>] [-AdminBg <RgbColor>]
  [-ErrorFg <RgbColor>] [-ErrorBg <RgbColor>]
  [<CommonParameters>]
```

note: This base command is pretty complex, and has a lot of parameters.
Passing all of those through is non-trivial, and I'd like to make it easier to contribute.
Let me show you what I came up with.

---

## What do _you_ ...
## Want to see more of?


---

note:
Let's say we wanted to write a module that supports being extended. For us, it's not Secret Management, but rather a module for configuration. We're going to let every module that has _settings_ be able to create named sets of their settings, so that users can easily switch between them. Maybe we'll even create modules to let us create sets of configuration for external apps, like Windows Terminal, or OBS, or for real world things like our house lighting, or the coffee maker...

note:
For now, let's say we want to switch our terminal from a "dark" background to a "light" background when we're doing a presentation, to get the best contrast for readability. We want to swap settings in Windows Terminal, PSReadLine and our prompt, and maybe even in the `$PSStyles` variable

note:
We could have a "dark" and a "light" configuration, and we could switch between them. We could also have a "work" and a "home" configuration, and switch between them. We could even have a "work" and a "home" configuration for each of the "dark" and "light" configurations, and switch between them. We could even have a "work" and a "home" configuration for each of the "dark" and "light" configurations for each of the "dark" and "light" configurations, and switch between them. You get the idea.

