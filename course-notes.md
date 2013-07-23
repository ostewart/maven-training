Intro to Maven Course Notes
===========================

## Why Maven?
### Regular Structure
* Maven turns source code into artifacts
* Not a general automation tool
  * Makes it really hard to do some things that lead to hidden dependencies and pain later
  * Also makes it really hard to do some pretty useful things
* Approachable for new developers
* Tools can understand your build
* Supports collective ownership–rarely see builds that only one person can touch
* Codifies good practice
  * Separate code, tests, resources
  * Declarative dependency management
  * Test final built artifacts
  * Simple build steps
  * Intricate resource copying is hard
  * Share resources via dependencies

  
Most projects are, at some level, doing the same thing: turning source code into build artifacts. They do this by running a compiler and running tests to verify the output (and input). There are a lot of other things associated with this, such as managing dependencies, replacing variable data in files, time and/or version stamping, etc., but nearly all builds have some of the same components and structure.

In the pre-Maven days, every build was a special snowflake. Depending on the build system author's mood or inclination, the location of source files, the specific compiler commands invoked, and how dependencies are managed would all be unique to each build. Any new developer trying to contribute to a project would first have to learn some of the intricacies of that build system before being productive. Maven takes all the common elements of a typical build and applies a predictable structure to them. This results in some degree of reduced flexibility in how you manage and lay out your software components, but it has the advantage that anyone familiar with a Maven build in general can very quickly understand and work with a specific project built with Maven. Another great benefit is that, because the structure is regular and machine-readable, tools can understand your build as well. Maven projects can easily be imported into a wide range of IDEs, and build automation tools such as Jenkins can trivially be setup to build and report on a Maven project without having to configure all the details such as test output locations or module structure.

#### Why Not Maven?
For some cases, Maven may not be the right tool. For general automation tasks that don't fit inside the box of turning source code into tested build artifacts, you're probably going to end up fighting Maven. Maven does have a plugin system that allows for many of the things you might need to do along the way to producing a build artifact. Many of the common tasks are already available as plugins. Alternatively, you can use the Maven AntRun plugin to invoke Ant scripts or wrap your Maven build with a shell or other script that performs the general automation tasks you need.

## The Maven Way

Maven has its way of doing things. If you have a problem that's not obvious to solve, first try to fit it into Maven's lifecycle and existing plugins.

## Maven Basics

### Creating a New Project
Start off with the simplest POM. Just a GAV.

### Maven Lifecycle
Default lifecycle:
* validate (basic project correctness)
* compile
* test (unit tests before packaging)
* package (build the output artifact e.g. jar or war)
* integration-test (more on this later)
* verify (quality checks)
* install (install artifact into local repository…most of the time you want this)
* deploy (not what you thing…deploys artifact to a Maven repository)

Each phase executes all the previous. Goals can be bound to phases. Packaging binds some initial goals to phases.

Jar:

Phase                 | Goal
----------------------|-------------------------
process-resources     | resources:resources
compile               | compiler:compile
process-test-resources| resources:testResources
test-compile          | compiler:testCompile
test                  | surefire:test
package               | jar:jar
install               | install:install
deploy                | deploy:deploy


We'll see more about binding goals to phases later. You can execute phases or goals directly on the command line.

### Basic Dependency Management
Maven manages transitive dependencies declaratively. This is one of the greatest benefits of Maven, and other tools (such as Ivy) have copied its model. A dependency consists minimally of a groupId, artifactId, and version. Optionally, a scope can be specified. Possible scopes are compile (the default), test, provided (added to classpath for compile, not runtime), runtime (not added to classpath at compile time), and system (don't use it…for specifying a local file).

### Directory Layout
Convention over configuration. Look at the Super POM. (mvn help:effective-pom)

### Super POM
All POMs inherit from it. Go there to see defaults. mvn help:effective-pom shows you your project with the Super POM values.

### Exercise: A Simple Library Project
Let's pretend we're building a Hello World library that prints Hello World out to the console. Write a HelloWorld class that outputs a String and a test that invokes it. Build and test your project with Maven. Solution in http://github.com/ostewart/maven-training

## More Basics
Now that we've seen how to create a very simple project built with defaults, let's look at something more complicated. Also note what we've gotten from just a few lines of XML as compared to what we'd have to do in Ant to handle compilation, test running, jar building, and version management.

### Plugins
Maven's main construct of modularity is the plugin. Many of the aspects of your project can be configured by configuring the plugins Maven uses to execute parts of the build. For example, the maven-compiler-plugin:

      <plugin>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>2.0.2</version>
        <configuration>
          <source>1.6</source>
          <target>1.6</target>
        </configuration>
      </plugin>

### Common Plugins
Plugins you'll frequently encounter include the compile plugin and the SureFire plugin that runs unit tests. Behind the scenes and able to be customized are the maven-resources-plugin, the maven-dependency-plugin, and others.

### Packaging
The packaging for a module defines the lifecycle bindings for that module as well as how the build artifact is packaged. Most common is jar packaging, that packages up your class files into a jar. Also common are the pom packaging, for container modules, and war packaging for web artifacts.

### Exercise: Add TestNG Unit Tests
Configure the SureFire plugin to run TestNG instead of JUnit.

## Multi-Module Maven
Most projects become complex enough that they can be decomposed into multiple modules. Maven's straightforward module system and transitive dependency system make it really easy to split your system into multiple modules (assuming you haven't already created crazy circular dependency issues).

A multi-module project typically consists of a top-level container with pom packaging and a modules section that references each module. Each module corresponds to a subdirectory of the parent project/module.

Modules are useful for grouping related functionality and, perhaps more importantly, for isolating functionality that should not be grouped. Strive for loose coupling and high cohesion. Vertical module stacks (service-web/webservice).

Digression: why separate code into modules? To enforce code boundaries, dependency relationships. As a unit for sharing code (can depend on a library jar). As a way of organizing dependencies. (E.g. some code needs an HTTP client. Not all your code should talk directly via HTTP. Isolate the HTTP client code into a module, then other code can depend on that without depending on the HTTP client.) As a unit of packaging (to generate a WAR, for example).