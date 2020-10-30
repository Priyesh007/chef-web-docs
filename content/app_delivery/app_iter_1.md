+++
title = "Define and Package App"
draft = false

[menu]
  [menu.app_delivery]
    title = "Define"
    identifier = "app_delivery/app_guide/app_iter_1"
    parent = "app_delivery/app_guide"
    categories = ["app_delivery"]
    weight = 50
+++
[\[edit on GitHub\]](https://github.com/chef/chef-web-docs/blob/master/content/app_delivery/app_iter_1.md)

In most cases, to deploy your [demo app] you would need to install Java SE and Tomcat on the machine where you want to run the app, download and move the `.war` file to the `CATALINA_HOME/webapps` directory, and then start the Tomcat service to deploy the application on port 8080.

With Chef App Delivery, you give instructions to Chef Habitat about what it needs to run, deploy, and manage the application. For the demo app, these instructions tell Chef Habitat to package up the `.war` file as part of the Habitat Artifact (`.HART`) and turns the entire deployment into a "hands-free" zone for the application. Once you package your app, the Chef Habitat Supervisor runs the it without human intervention. When a you need to update your app, you'll package up a new version and create a new `.HART` file that Chef Habitat automatically passes to the Supervisor.

This guide assumes that you have access to an internet connection and the `bldr.habitat.sh` domain.

You'll define your App from inside the Chef Habitat Studio. Since Chef Habitat only loads what necessary into the Studio, the initial installation consists of the `hab` CLI. Chef Habitat downloads additional components into the Studio as you need them. The first time you run `hab studio enter`, the command will take a few seconds while it acquires the packages it needs to load the Studio environment.

Start the Chef Habitat Studio from inside the `java-sample-master` directory.

```bash
cd java-sample-master
hab studio enter
```

## Define your App

The plan file ties together the Habitat Manifest. The plan file defines important metadata about your application like it's name and version, and contains the definition for how the app should be packaged for delivery.

The `hab plan init ` command created `plan.sh` file in the `habitat` director. The plan file is pre-populated with metadata fields and a number of functions, which are "turned off" with code comments. You can "turn on" the functions by removing the hash marks (`#`). The plan file includes links to the Chef Habitat documentation, where you can learn more about writing plans.

When you start working on your plan, you'll notice it has a `.sh` extension, which means that it is a Bash script. One of the ways that Chef Habitat removes the "noise" between your app and your automation is by using using common file formats. Plans for Linux applications use Bash and plans for Windows applications use Powershell. You don't need to be a Bash expert for this walkthrough, but understanding a few concepts will help you go faster.

### Start with a Simple Plan

You don't need everything included in the default plan file, but you do need to keep the basic metadata fields and the plan functions, which you'll learn more about soon. To start off, delete everything in your `plan.sh ` file except for the following:

```bash
pkg_name=java-sample
pkg_origin=INITIALS_tryhab
pkg_version="0.1.0"
pkg_maintainer="The Habitat Maintainers <humans@habitat.sh>"
pkg_license=("Apache-2.0")
pkg_shasum="TODO"
pkg_deps=(core/glibc)
pkg_build_deps=(core/make core/gcc)

do_unpack() {
  do_default_unpack
}

do_build() {
  do_default_build
}

do_install() {
  do_default_install
}
```

## App Metadata

Your plan uses simple Bash variables for the plan metadata, which is the information about the package that you're building. Chef Habitat reserves these names and uses them internally for various tasks. The metadata variables in this example are: `pkg_name`,`pkg_origin`, `pkg_version`,`pkg_maintainer`, `pkg_license`, and `pkg_shasum`. You'll define most of these yourself, but some, for example `pkg_shasum` are generated by Chef Habitat. You set the metadata variable by providing a value. For example, to set `pkg_name`:

```bash
pkg_name=java-sample
```

The metadata variable `pkg_name` now has the value `java-sample`. This is a normal bash variable and you can reuse it later in your `plan.sh` by calling it with the format `${varible}`. For example, reuse the `pkg_name` name with:

```bash
${pkg_name}
```

Another common variable for use within a Plan is `${PREFIX}`, which contains anything that should be packaged into the resulting binary. During the packaging phase Chef Habitat replaces `${PREFIX}` with the dynamically generated artifact name, which by default appends the date timestamp from when the package was built.

## App Dependencies

Apps have three types of dependencies: buildtime, runtime, and transitive. You'll declare the buildtime and runtime dependencies in your plan and Chef Habitat will automatically handle the transitive dependencies.

There are no standard dependencies for an package. Dependencies come from what your application needs to have when build or run it.  For example, `coreutils` is a common buildtime dependency that is usually not needed at runtime, and `glibc` is a common runtime dependency that is not needed at buildtime. However, your dependencies obviously depend on the needs of your application.

**Buildtime dependencies** are available within the Studio and are needed to package the application. Declare them with `pkg_build_deps`.

**Runtime dependencies** are shipped with the binary and are needed to run the application. Declare them with `pkg_deps`.

**Transitive Dependencies** are installed as dependencies of buildtime `pkg_build_deps` or runtime `pkg_deps` and do not need to be explicitly declared. You will see them during the build process and in the Studio `/hab/pkgs/` directory.

### Core Packages

Chef Habitat provides many commonly used software dependencies packages in the ["core"](https://bldr.habitat.sh/#/origins/core/packages) origin. You can search for [core packages on the Chef Habitat Builder](https://bldr.habitat.sh/#/pkgs/core) site.

Your current plan file uses core packages in its description of runtime and buildtime dependencies.

```bash
pkg_deps=(core/glibc)
pkg_build_deps=(core/make core/gcc)
```

### Update your Dependencies

The demo app needs the Java Runtime Environment 8, and Tomcat8 software packages to run. Both of these are available as core packages.

Update your plan to describe the app dependencies:

```bash
pkg_deps=(core/tomcat8 core/corretto11)
```

## App Build Instructions

Chef Habitat uses two types of functions in describing an app build phase: _build phase callbacks_ and _plan helper functions_. You only need build phase callbacks for your demo app. Plan helper functions are bash functions that use inside of build phase callbacks to perform specific tasks. You can learn more about them in the [plan helpers] documentation.

### Build Phase Callbacks

Build phase callbacks are are standard Bash functions that Chef Habitat reserves for using during build processes. Build phase callbacks make your code more readable and easier to use.

There are a number of functions that correspond to each build phase. For example, the `do_build()` function builds your applications from source, and uses make by default to build an executable binary. If you're not building from source or your application can't be built with `make` then you'll put the appropriate build commands into body of this function.

Sometimes you will want to skip a function, for for example, if you aren't building your app from source you won't need to use the `do_build()` function. To skip a function, return a zero status in the body of the function:

```bash
do_build() {
  return 0
}
```

After the source components are available, you can use `do_install()` to be used to move them the right places for packaging. Some components might only be needed during the build process and might be unnecessary afterwards.

#### Update your App Build Instructions

You'll use the build the demo app with local `.war` file from sample-app directory, which means that you can return 0 (skip) the `do_unpack()` and `do_build()` steps, because you don't need to expand the application or build it with `make`. Because you already have the `.war` file available locally, you can package it with the binary and install it with `do_install()`. In this step, you'll use `do_install()` to copy the `.war` file to the `${PREFIX}` directory.

Change your build instructions:

```bash
do_unpack() {
  return 0
}

do_build() {
  return 0
}

do_install() {
  cp ${pkg_name}.war ${PREFIX}/
}
```

## Your Demo App Plan

At this point, your demo app plan should look like:

```bash
pkg_name=java-sample
pkg_origin=INITIALS_tryhab
pkg_version="0.1.0"
pkg_maintainer="The Habitat Maintainers <humans@habitat.sh>"
pkg_license=("Apache-2.0")
pkg_shasum="TODO"
pkg_deps=(core/tomcat8 core/corretto11)

do_unpack() {
  return 0
}

do_build() {
  return 0
}

do_install() {
  cp ${pkg_name}.war ${PREFIX}/
}
```

Save this file, and it's time to build and package the application!