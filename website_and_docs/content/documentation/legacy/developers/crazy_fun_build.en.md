---
title: "Crazy Fun Build Tool"
linkTitle: "Crazy Fun"
weight: 6
---

(Previously located: https://github.com/SeleniumHQ/selenium/wiki/Crazy-Fun-Build/)

WebDriver is a large project: if we tried to push everything into a single monolithic build file it eventually becomes unmanageable. We know this. We've tried it. So we broke the single Rakefile into a series of `build.desc` files. Each of these describe a part of the build.

Let's take a look at a build.desc file. This is part of the [main test build.desc](https://github.com/SeleniumHQ/selenium/blob/master/java/client/test/org/openqa/selenium/build.desc):

```
java_test(name = "single",
  srcs = [
    "SingleTestSuite.java",
  ],
  deps = [
    ":tests",
    "//java/server/src/org/openqa/selenium/server",
    "//java/client/test/org/openqa/selenium/v1:selenium-backed-webdriver-test",
    "//java/client/test/org/openqa/selenium/firefox:test",
  ]  ])

```

## Targets

This highlights most of the key concepts. Firstly, it declares **target**, in this case there is a single `java_test` target.
Each target has a `name` attribute.

### Target Names

The combination of the location of the "build.desc" file and the name are used to derive the rake tasks that are generated. All task names are prefixed with "//" followed by the path to the directory containing the "build.desc" file relative to the `Rakefile`, followed by a ":" and then the name of the target within the "build.desc". An example makes this far clearer :)

The rake task generated by this example is `//java/client/test/org/openqa/selenium:single`

### Short Target Names

As a shortcut, if a target is named after the directory containing the "build.desc" file, you can omit the part of the rake task name after the colon. In our example: `//java/server/src/org/openqa/selenium/server` is the same as `//java/server/src/org/openqa/selenium/server:server`.

### Implicit Targets

Some build rules supply implicit targets, and provide related extensions to a normal build target. Examples include generating archives of source code, or running tests. These are declared by appending a further colon and the name of the implicit target to the full name of a build rule. In our example, you could run the tests using "//java/client/test/org/openqa/selenium:single:run"

Each of the rules described below have a list of implicit targets that are associated with them.

### Outputs

Each target specified in a "build.desc" file produces one and only one output. This is important. Keep it in mind. Generally, all output files are placed in the "build" directory, relative to the rake task name. In our example, the output of "//java/org/openqa/selenium/server" would be found in "build/java/org/openqa/selenium/server.jar". Build rules should output the names and locations of any files that they generate.

### Dependencies

Take a look at the "deps" section of the "single" target above. The `":tests"` is a reference to a target in the current "build.desc" file, in this case, it's a "java\_library" target immediately above. You'll also see that there's a reference to several full paths. For example `"//java/server/src/org/openqa/selenium/server"` This refers to another target defined in a crazy fun build.desc file.

### Browsers

The py\_test and js\_test rules have special handling for running the same tests in multiple browsers.  Relevant browser-specific meta-data is held in rake-tasks/browsers.rb.  The general way to use this is to append `_`browsername to the target name; without the `_`browsername suffix, the tests will be run for all browsers.

As an example, if we had a js\_test rule //foo/bar, we would run its tests in firefox by running the target //foo/bar\_ff:run or we would run in all available browsers by running the target //foo/bar:run

## Build Targets
You can list all the build targets using the `-T` option. e.g.
```
./go -T
```

Being a brief description of the available targets that you can use.

### Common Attributes

The following attributes are required for all build targets:

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| name               | string   | Used to derive the rake target and (often) the name of the generated binary |

The following attributes are commonly used:

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| srcs               | array    | The raw source to be build for this target |
| deps               | array    | Prerequisites of this target |

### java\_library

* **Output:** JAR file named after the "name" attribute if the "srcs" attribute is set.
* **Implicit Targets:** run (if "main" attribute specifiec), project, project-srcs, uber, zip
* **Required Attributes:** "name" and at least one of "srcs" or "deps".

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| deps               | array    | As above    |
| srcs               | array    | As above    |
| resources          | array    | Any resources that should be copied into the jar file. |
| main               | string   | The full classname of the main class of the jar (used for creating executable jars) |

### java\_test

* **Output:** JAR file named after the "name" attribute if the "srcs" attribute is set.
* **Implicit Targets:** run, project, project-srcs, uber, zip
* **Required Attributes:** "name" and at least one of "srcs" or "deps".

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| deps               | array    | As above.   |
| srcs               | array    | As above.   |
| resources          | array    | Any resources that should be copied into the jar file. |
| main               | string   | The alternative class to use for running these tests. |
| args               | string   | The argument line to pass to the main class |
| sysproperties      | array    | An array of maps containing System properties that should be set |

### js\_deps

* **Output:** Marker file to indicate task is up to date.
* **Implicit Targets:** None
* **Required Attributes:** "name" and "srcs"

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| name               | string   | As above    |
| srcs               | array    | As above    |
| deps               | array    | As above    |

### js\_binary

* **Output:** A monolithic JS file containing all dependencies and sources compiled using the closure compiler without optimizations.
* **Implicit Targets:** None
* **Required Attributes:** At least one of srcs or deps.

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| name               | string   | As above    |
| srcs               | array    | As above    |
| deps               | array    | As above    |

### js\_fragment

* **Output:** Source of an anonymous function representing the exported function, compiled by the closure compiler with all optimizations turned on.
* **Implicit Targets:** None
* **Required Attributes:** name, module, function, deps

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| name               | string   | As above    |
| module             | string   | The name of the module containing the function |
| function           | string   | The full name of the function to export |
| deps               | array    | As above    |

### js\_fragment\_header

* **Output:** A C header file with all js\_fragment dependencies declared as constants.
* **Implicit Targets:** None
* **Required Attributes:** name, deps

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| name               | string   | As above    |
| srcs               | array    | As above    |
| deps               | array    | As above    |

### js\_test

* **Output:**
* **Implicit Targets:** `_`BROWSER:run, run
* **Required Attributes:** None.

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| deps               | array    | As above.   |
| srcs               | array    | As above.   |
| path               | string   | The path at which to expect the test files to be hosted on the test server. |
| browsers           | array    | List of browsers, from rake\_tasks/browsers.rb, to run the tests in.  Will only attempt to run tests in those browsers which are available on the system.  If absent, defaults to all browsers on the system. |

Assuming browsers = ['ff', 'chrome'], for target //foo, the implicit targets: //foo\_ff:run and //foo\_chrome:run will be generated, which run the tests in each of those browsers, and the implicit target //foo:run will be generated, which runs the tests in both ff and chrome.

### py\_test

* **Output:** Creates the directory structure required to run the listed python tests.
* **Implicit Targets:** `_`BROWSER:run, run
* **Required Attributes:** name.

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| deps               | array    | Other py\_test rule(s), whose tests should also be run. |
| common\_tests      | array    | Test file(s) to be run in all browsers.  These tests will be passed through a template, with browser-specific substitutions, so that they are laid out properly for each browser in the python output file tree. |
| BROWSER\_specific\_tests | array    | Test file(s) to be run only in browser BROWSER. |
| resources          | array    | Resources which should be copied to the python directory structure. |
| browsers           | array    | List of browsers, from rake\_tasks/browsers.rb, to run the tests in.  Will only attempt to run tests in those browsers which are available on the system.  If absent, defaults to all browsers on the system. |

Note: Every py\_test invocation is performed in a new virtualenv.

### rake\_task

* **Output:** A crazy fun build rule that can be referred to "blow the escape" hatch and use ordinary rake targets.
* **Implicit Targets:** None
* **Required Attributes:** name, task\_name, out.

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| name               | string   | As above    |
| task\_name         | string   | The ordinary rake target to call |
| out                | string   | The file that is generated, relative to the Rakefile |


### gcc\_library

* **Output:** Shared library file named after the "name" attribute if the "srcs" attribute is set.
* **Implicit Targets:** None.
* **Required Attributes:** "name" and "srcs".

| **Attribute Name** | **Type** | **Meaning** |
|:-------------------|:---------|:------------|
| srcs               | array    | As above    |
| arch               | string   | "amd64" for 64-bit builds, "i386" for 32-bit builds. |
| args               | string   | Arguments to the compiler (-I flags, for example). |
| link\_args         | string   | Arguments to the linker (-l flags, for example) |

Note: When building a new library for the first time, the build will succeed but copying to pre-built will fail with a similar message:

```
cp build/cpp/amd64/libimetesthandler64.so 
go aborted!
can't convert nil into String
```

Solution: Copy the just-built library to the appropriate prebuilt folder (cpp/prebuilt/arch/).