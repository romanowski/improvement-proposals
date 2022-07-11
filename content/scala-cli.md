---
layout: sip
permalink: /sips/:title.html
stage: implementation
status: waiting-for-implementation
title: SIP-NN Scala CLI as default Scala command
---

**By: Krzysztof Romanowski and  Scala CLI team**

## History

| Date          | Version            |
|---------------|--------------------|
| July 1th 2022 | Initial Draft      |

## Summary

We propose to replace current scripts that are installed as "Scala": scala, scalac and scaladoc with scala-cli - a batteries included tool to interact with Scala. Scala CLI brings all the features that the command above provide but expanding it with incremental compilation, dependency handling, packaging and more. 


## Motivation

The current default Scala scripts: `scala`, `scalac`, `scaladoc` are quite limited. Beside `scala` command that starts repl are hardly used by non-power users. 

The current scripts are lacking such a basics features like support for resolving dependencies, incremental compilation or support for outputs other then JVM. This forces any user that wants to do anything more then just basic things to learn and use SBT, mill or other build tool and that adds to the complexity of learning Scala. 

We observe that current state of tooling is Scala is limiting the creativity, with quite high cost to create e.g. an application or script with some dependencies that targets node.js. Many Scala developers are not choosing Scala for their personal project, scripts or small applications and we believe that complexing of setting up a build tool is one of the reasons. 

With this proposal our main goal is to turn Scala in a language with 'batteries included' that will also respect community-first aspect of our ecosystem.

---------

A high-level overview of the proposal with:

- An explanation of the problems or limitations that it aims to solve,
- A presentation of one or more use cases as running examples, with code showing how they would be addressed *using the status quo* (without the feature), and why that is not good enough.

This section should clearly express the scope of the proposal. It should make it clear what are the goals of the proposal, and what is out of the scope of the proposal.

## Proposed solution

We propose to gradually replace current `scala`, `scalac` and `scaladoc` commands by single `scala` command that under the hood will be `scala-cli`. We could also add wrapper scripts for `scalac` and `scaladoc` that will mimic the functionality that will use scala-cli under the hood. 

The complete set of `scala-cli` features can be found in [the documentation](https://scala-cli.virtuslab.org/docs/overview).

Scala CLI brings many features like testing, packaging, exporting to sbt/mill or upcoming support for packaging micro-libraries. Initially, we may want to limit the set of features available in the `scala` command by default. Scala CLI is relatively new project and we should battle-proof some of the feature before we commit to support it as part of offical `scala` command. 

 Scala CLI offers a [multiple native ways to be installed](https://scala-cli.virtuslab.org/install#advanced-installation) so most users should find a suitable method. We would like to migrate this set to be default `scala` package, often replacing existing packages.



--- 
This is the meat of your proposal.

### High-level overview

Let us show a few examples where adopting Scala CLI as 'scala' command would be an significant improvement ofer current Scripts. For this, we have assumed a minial set of feauters. Each additnial  Scala CLI feature included such as `package` would add more and more usecases.

**Using REPL with a 3-party dependency**

Currently, to start repl with a dependency on a classpath users needs to resolve those depenendncy with all its treansitive dependencies (coursier can help here) and pass it to `scala` command using `--cp` option. Alternatively, one can create a sbt project including single dependency and use `sbt console` task. Ammonite gives the best experience with its magic imports. 

With Scala CLI starting a repl with a given depenency is as simple as starting:

```scala-cli repl --dep com.lihaoyi::os-lib:0.7.8```

Compared to ammonite, default repls provided by Scala 2 and 3 that Scala CLI uses by default are somewhat limited however Scala CLI also intergrate with ammonite and starting ammonite instead of default repl is as simple as passing `--ammonite` (or `--amm`) option to repl.

Additinally, Scala CLI repl can also put code from given files / directories / snippets on the classpath by just providing location as arguments. Running ```scala-cli repl foo.scala baz``` will compile code from `foo.scala` and `baz` directory and put output on repl classpath (including all dependencies, options etc. defined within those files).

Compilation (and doc as well) benefit in the similar way from ability manage dependencies.

** Poroving reproduction of the bug **

Currently, when a bug raported in compiler (or any other Scala-related) repository users needs to provide used depencencies, compiler options etc. in comments, create a repository containing a projet with mill/sbt configuration to reproduce. In general, testing the reporoduction or working on further minization is not straightworwad.

Using directives, provided by Scala CLI gives the ablity to include the whole configuration in single fine, for example:

```scala
//> using platform "native"
//> using "com.lihaoyi::os-lib:0.7.8"
//> using options "-Xfatal-warnings"
 
def foo = println("<here comes the buggy warning with Scala Native and os-lib>")
```

The snippet above when run with Scala CLI without any configuration provided will use Scala Native, resolve and include `os-lib` and provide `-Xfatal-warnings` to compiler. Even such things as runtime JVM version can be cofigured with Using Directieves.

Moreover, Scala CLI provide ability to run gists (also multi-file) and more.

** Watch mode **

When working on a pice of code, often it is useful to have it compiled/run everytime file is changed and build tools offera watch mode for that. This is how most people are using - through a build tool. Scala CLI offers watch more for most commands (by using `--watch` / `-w` flags).

// TODO more?

-------

A high-level overview of the proposed changes, and how they allow to better solve the running examples. This section should be example-heavy, and not dive into corner cases.

Example:

~~~ scala
// This is an @main method
@main def foo(x: Int): Unit =
  println(x)
~~~

### Specification

 In order to be able to expand the functionality of scala-cli and yet use the same core to power `scala` command we propose to included both `scala` and `scala-cli` commands in the installation package. Scala CLI already has a feature to limit accessible commands based the used binary (all commands in `scala-cli` and curated list in the `scala` command). Later, we can include more and more feature from `scala-cli` into `scala`. 

The commands necessary to include in the `scala` to match the functionaries of current commands:
 - `compile`
 - `run`
 - `repl`
 - `doc`

On top of that, we thing that following using-facing commands should be also included:
 - `clean` - to rebuild project from start without any cached steps
 - `setup-ide` - to control setup for BSP for IDEs integration
 - `doctor` - to analyze if everything is installed properly

We also suggest to include additional commands by default: 
 - `fmt` - to format the code using scalafmt
 - `test` - to run test incuded in the code
 - `package` - to package the code into various projects: jar, fat-jar, executable jar or even native application or docker image
 - `shebang` - a command useful for scripting, designed to be incuded in `shebang` section of the scripts
 - `export` - transform current project to `sbt`/`mill`project using the same settings as provided. Useful to evolve prototypes into bigger projects

Each of those commands expand what current scripts offers and can be discussed severalty. We can even open a dedicated SIP for each of them.

Beyond that, `scala-cli` offers multipe commands needed to manage itself (e.g. update) or its components (e.g. Bloop server) that in most cases are not user-facing but needs to be available. We can elaborate in more details what are those commands and why we need them if needed.


Scala CLI can be also configured using [using directives](https://scala-cli.virtuslab.org/docs/guides/using-directives) - a comment based configuration syntax that should be placed on top of Scala files. This allows for self-containing examples withing one file since most configuration can be provided either from command line or using directives (command line has precedence). This is a game changer for surceases like scripting, reproductions or within education scope.

We have described the motivation, syntax and implementation basis in the [dedicated pre-SIP](https://contributors.scala-lang.org/t/pre-sip-using-directives/5700). Currently, we recommend to place using directives in the comments so making them part of the language specification is not necessary at this sage. Moreover, the new `scala` command could ignore using directives in the initial version, however we strongly suggest to include comment-based using directives from the beginning. 

---

A specification for the proposed changes, as precise as possible. This section should address difficult interactions with other language features, possible error conditions, and corner cases as much as the good behavior.

For example, if the syntax of the language is changed, this section should list the differences in the grammar of the language. If it affects the type system, the section should explain how the feature interacts with it.

### Compatibility

Adopting Scala CLI as new `scala` command as is will change some behaviours of current scripts. Some examples:

- `scala repl`  needs to run to start `repl` instead of `scala`
- with `scala compile` and `scala doc` some of the more obscure (or brand new) compile options will needs to be prefixed with `-o`
- Scala CLi recognizes tests based on the extension used (`*.test.scala`) so running `scala compile a.scala a.test.scala` will only compile `a.scala`
- Scala CLI has its own versioning scheme, that is not related to Scala compiler. Default version used may dynamically change when new Scala version is released.
- By default, Scala CLI manages its own dependencies (e.g. scalac, zinc, bloop) and resolved them lazily. This means that first run after Scala CLI is installed will resolve quite some dependencies. Moreover, Scala CLI periodically check for updates, new defaults accessing online resources (but it is not required to work, so Scala CLI can work in offline environment once setup)
- Scala CLI can be also configured using driectives. Command line options has precendece over using directives, however using directives overrides the default. Compiling file starting with following line `//> using scala 2.13.8` without provided version in command line will result using `2.13.8` rather then default version. We consider this a feature however technially is a breaking change.

// TODO more?

---

A justification of why the proposal will preserve backward binary and TASTy compatibility. Changes are backward binary compatible if the bytecode produced by a newer compiler can link against library bytecode produced by an older compiler. Changes are backward TASTy compatible if the TASTy files produced by older compilers can be read, with equivalent semantics, by the newer compilers.

If it doesn't do so "by construction", this section should present the ideas of how this could be fixed (through deserialization-time patches and/or alternative binary encodings). It is OK to say here that you don't know how binary and TASTy compatibility will be affected at the time of submitting the proposal. However, by the time it is accepted, those issues will need to be resolved.

This section should also argue to what extent backward source compatibility is preserved. In particular, it should show that it doesn't alter the semantics of existing valid programs.

### Other concerns

Scala CLI brings [using directives](https://scala-cli.virtuslab.org/docs/guides/using-directives) and  [conventions to mark the test files](https://scala-cli.virtuslab.org/docs/commands/test#test-sources). We are not sure if both can be accepted as a part of this SIP or we should have seperate SIPs for both (we have open a [pre-SIP for using directives](https://contributors.scala-lang.org/t/pre-sip-using-directives/5700/15))

The Scala CLI is ambitious project and may seem hard to maintain in long-run. // TODO

----

If you think of anything else that is worth discussing about the proposal, this is where it should go. Examples include interoperability concerns, cross-platform concerns, implementation challenges.


### Open questions

The main open question for this proposal is wich commands/features should be included by default in the `scala` command. 

-----

If some design aspects are not settled yet, this section can present the open questions, with possible alternatives. By the time the proposal is accepted, all the open questions will have to be resolved.

## Alternatives

Scala CLI has many alternatives. The most obvious ones are `sbt`, `mill` or other build tools however they are more complicated then Scala CLI and what is more important they are not designed as command-line first tools. Ammonite, is another alternative, however it covers only part of the Scala CLI features (repl and scripting). 

---

This section should present alternative proposals that were considered. It should evaluate the pros and cons of each alternative, and contrast them to the main proposal above.

Having alternatives is not a strict requirement for a proposal, but having at least one with carefully exposed pros and cons gives much more weight to the proposal as a whole.

## Related work

- [Scala CLI website](https://scala-cli.virtuslab.org/) and [road map](https://github.com/VirtusLab/scala-cli/discussions/1101)
- [Pre-SIP](https://contributors.scala-lang.org/t/pre-sip-scala-cli-as-new-scala-command/5628/22)
- [leiningen](https://leiningen.org/) - a similr tool from Closure, but more configuration-oriented

------

This section should list prior work related to the proposal, notably:

- A link to the Pre-SIP discussion that led to this proposal,
- Any other previous proposal (accepted or rejected) covering something similar as the current proposal,
- Whether the proposal is similar to something already existing in other languages,
- If there is already a proof-of-concept implementation, a link to it will be welcome here.

## FAQ

This section will probably initially be empty. As discussions on the proposal progress, it is likely that some questions will come repeatedly. They should be listed here, with appropriate answers.
