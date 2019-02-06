# Writing a Serverless Saga - in code

In this workshop, you're going to create a serverless implementation of the [saga pattern](https://microservices.io/patterns/data/saga.html).

You're going to build this using [Fn](https://fnproject.io/), an open source, container - native, serverless platform.

## Agenda
1.  [Introduction to Fn](#introduction)
2.  [Install Fn](#install)
3.  [Creating functions](#create)
4.  [Group functions together in apps](#group)
5.  [Orchestration with Flow](#orchestrate)
6.  [Implement a Saga with Fn and Flow](#saga)

## Pre - requisites
This workshop requires a Docker environment running on Linux (or Mac).

The assumption is that you have one of...
- Linux
- Mac
- Linux VM on Windows

...with Docker installed

## <a name="introduction"/> Introduction to Fn

## <a name="install"/> Install Fn
To install Fn, we first install the CLI and then use that to "install" the server (which actually runs as a Docker container).

To download and install the Fn CLI, follow these [instructions](http://fnproject.io/tutorials/install).

Then start the Fn server:

`fn start`

and test it.

You will need to use two ssh sessions here, one to run the server, the other to interact with it.

If you prefer you can run the server in the background by using the -d flag:

`fn start -d`.

In this case you won't see the startup text, but will see the docker container id.

If having started the server you decide to move it to the background you can stop the server with Ctrl-C or run the command `fn stop` in another window, then re-start it with `fn start -d` (If you restart it it doesn't need to download the fn server docker images, so it's much faster on subsequent starts)

If you do start the server in the background be sure to have it running when you start configuring the context.

As we're using the script to setup Fn you can skip the "Fn Manual Install" section.

Once you've got your Fn server up and running, you should be ready to create your first function.

## <a name="create"/> Creating functions
Fn uses Docker containers as function primitives.  This means that you can write functions in any language that can run in a Docker container.  You can even run a shell script as a function if you like.

However the easiest way to create a function is with the help of an FDK (Function Development Kit).  The FDK takes care of:
- reading input data
- writing output data
- writing errors
- handling repeat invocations (of the same function instance)

The last part is necessary because Fn functions are "hot" - once the function has executed, the container runs for a short (configurable) time and can be reused if the function is invoked again during that time.

If you don't use an FDK, you need to take care of these yourself.

### Using an FDK
To create a function using one of the FDKs, click on the link below to go to a tutorial for the language of your choice:

- [Go](http://fnproject.io/tutorials/Introduction/)
- [Java](http://fnproject.io/tutorials/JavaFDKIntroduction/)
- [Node.js](http://fnproject.io/tutorials/node/intro/)
- [Python](http://fnproject.io/tutorials/python/intro/)
- [Ruby](http://fnproject.io/tutorials/ruby/intro/)

If you like you can try creating functions using multiple FDKs.  They all work the same way and since the functions are isolated, your applications can be include functions written in any number of languages.  

### From a Docker image (optional)
You can also [create a function using a Docker image](http://fnproject.io/tutorials/ContainerAsFunction/).

### Using HotWrap (optional, *warning - experimental feature!!!*)
HotWrap is an experimental tool that enable you to turn any shell command into a function.  It is essentially and FDK that sends incoming events to your command as STDIN and reads the output on STDOUT.

If you're interested you can try it out [here](https://github.com/fnproject/hotwrap) (but again, it is experimental).

## <a name="group"/> Group functions together in applications (apps)

An Fn function is always deployed as part of an `application`.  When you deploy even a single function you do this within an application.

When you list deployed functions you do this by application e.g. `fn list functions myapp`.

When an app consists of multiple functions, these can be deployed in a single operation, rather than individually.

Configuration values can also be specified at the level of an application, and thus applied to all of the functions in the app, as opposed to setting the values for the individual functions.

## <a name="orchestrate"/> Orchestration with Flow

As you've seen, applications can be used to organise function code, deployment and configuration.

In the vast majority of cases, performing useful work for a user will require multiple functions to be invoked.  These invocations will typically need to be coordinated to make sure that they happen in the correct sequence.  In other words you need to manage the application workflow.

Fn uses [Flow](https://github.com/fnproject/flow) as it's orchestration mechanism to manage the application workflow across multiple functions.

Flow uses a promises style API written in a normal programming language (rather than an XML or YAML dialect) to define workflows (each called a flow).

This API is currently only implemented in Java but other are implementations on the way.

The first two flow examples are:
- [Flow 101](http://fnproject.io/tutorials/Flow101/)
- [Flow 102](http://fnproject.io/tutorials/Flow102/)


The functions invoked by the flow can be written in any language that you choose.

Since the flow function is written in code, you could put embed other operations into it, but the best advice is to take a "separation of concerns" approach and to use the flow purely for orchestration, and deliver the business functionality via the invoked functions.

## <a name="saga"/> Implement a Saga with Fn and Flow 

## Beyond Hello World
Once you've got your Hello World! function running let's see how Fn can group multiple functions together to create an application (OK, this is a *very* simple application, it just responds to requests on two different url's).

One interesting point here is that the functions are in multiple languages, but because of the runtime compartmentalisation the Fn server provides they still run happily together as a single app.

If you want to explore more have a look at the various [FDK's (Function Developer Kit)](https://github.com/fnproject).

There is a range of examples, for example [generating QR codes](https://github.com/fnproject/fdk-java/tree/master/examples/qr-code).