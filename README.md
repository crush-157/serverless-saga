# Writing a Serverless Saga - in code

In this workshop, you're going to create a serverless implementation of the [saga pattern](https://microservices.io/patterns/data/saga.html).

You're going to build this using [Fn](https://fnproject.io/), an open source, container - native, serverless platform.

## Agenda
1.  [Introduction to Fn](#introduction)
2.  [Install Fn](#install)
3.  [Creating functions](#create)
4.  [Logging and Troubleshooting](#log)
5.  [Group functions together in apps](#group)
6.  [Orchestration with Flow](#orchestrate)
7.  [Implement a Saga with Fn and Flow](#saga)

---

## Pre - requisites
This workshop requires a Docker environment running on Linux (or Mac).

The assumption is that you have one of...
- Linux
- Mac
- Linux VM on Windows

...with Docker installed

---

## <a name="introduction"/> Introduction to Fn

### Overview
[Fn](https://fnproject.io) is an open - source, container-native serverless platform.

The core is written in Go, but Fn supports functions written in any language (as long as it runs in a Docker container).

Licensed under the Apache License 2.0.

Code available on [GitHub](https://github.com/fnproject).

![](images/slides/fn-project.png)

![](images/slides/functions-as-containers.png)

![](images/slides/an-fn-function.png)

#### Hello World functions:

##### Java

```
package com.example.fn;

public class HelloFunction {

    public String handleRequest(String input) {
        String name = (input == null || input.isEmpty()) ? "world"  : input;

        return "Hello, " + name + "!";
    }

}
```

##### Ruby (func.rb)
```
require 'fdk'

def myfunction(context:, input:)
  input_value = input.respond_to?(:fetch) ? input.fetch('name') : input
  name = input_value.to_s.strip.empty? ? 'World' : input_value
  { message: "Hello #{name}!" }
end

FDK.handle(target: :myfunction)
```

##### Function Metadata (func.yaml):
```
schema_version: 20180708
name: hello
version: 0.0.1
runtime: ruby
entrypoint: ruby func.rb
format: http-stream
triggers:
- name: hello
  type: http
  source: /hello
```

A __function__ is deployed to an __app__ using the Fn CLI:

_Note:_

_Due to a recent change to the CLI, you need to create your app __before__ you can deploy to it.  The tutorials may not have all caught up yet!:_

To create the app:

`fn create app my-app`

To deploy the function:

`fn deploy --app my-app`

![](images/slides/fn-deploy.png)

The fn CLI is your friend ;-)

To get help, follow the usual pattern of "stick the `--help` flag after your command", e.g. `fn --help` or `fn list --help`

![](images/slides/endpoints.png)

![](images/slides/triggers.png)

We can invoke a function either by
- using the CLI (e.g. `fn invoke my-app my-function`)
- HTTP request to
  - trigger URL
  - default endpoint URL

![](images/slides/architecture.png)

![](images/slides/fn-server.png)

### Request Processing
![](images/slides/request-process.png)

---

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

---

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

If you like you can try creating functions using multiple FDKs.  They all work the same way and since the functions are isolated, your applications can include functions written in any number of languages.  

### From a Docker image (optional)
You can also [create a function using a Docker image](http://fnproject.io/tutorials/ContainerAsFunction/).

### Using HotWrap (optional, *warning - experimental feature!!!*)
HotWrap is an experimental tool that enable you to turn any shell command into a function.  It is essentially and FDK that sends incoming events to your command as STDIN and reads the output on STDOUT.

If you're interested you can try it out [here](https://github.com/fnproject/hotwrap) (but again, it is experimental).

---

## <a name="log"/> Logging and Troubleshooting

When something goes wrong you want to know about it.

To get more detail on what happens when you run an fn command, use the `--verbose` flag, e.g. `fn --verbose build`

Once your function is deployed, if it throws an exception, this will be written to STDERR by the FDK and then to syslog by the Fn server.

By the same token, if your function writes to STDERR this will be written to the log.

To collect and view the logs you need to configure a syslog URL for your app.

You can do this either
- for a new app:

  `fn create app kraftwerk --syslog-url tcp://logs7.papertrailapp.com:11125`

- or an existing app:

  `fn update app bauhaus --syslog-url tcp://logs7.papertrailapp.com:11125`

See the [troubleshooting tutorial](http://fnproject.io/tutorials/Troubleshooting/) for a more detailed description of how to set up logging.

---

## <a name="group"/> Group functions together in applications (apps)

An Fn function is always deployed as part of an `application`.  When you deploy even a single function you do this within an application.

When you list deployed functions you do this by application e.g. `fn list functions myapp`.

When an app consists of multiple functions, these can be deployed in a single operation, rather than individually.

Configuration values can also be specified at the level of an application, and thus applied to all of the functions in the app, as opposed to setting the values for the individual functions.


To understand how to use apps to group functions, work through [this example](http://fnproject.io/tutorials/Apps/).

---

## <a name="orchestrate"/> Orchestration with Flow

As you've seen, applications can be used to organise function code, deployment and configuration.

In the vast majority of cases, performing useful work for a user will require multiple functions to be invoked.  These invocations will typically need to be coordinated to make sure that they happen in the correct sequence.  In other words you need to manage the application workflow.

Fn uses [Flow](https://github.com/fnproject/flow) as it's orchestration mechanism to manage the application workflow across multiple functions.

Flow uses a promises style API written in a normal programming language (rather than an XML or YAML dialect) to define workflows (each called a flow).

This API is currently only implemented in Java but other are implementations on the way.

The first two flow examples you should work through are:
- [Flow 101](http://fnproject.io/tutorials/Flow101/)
- [Flow 102](http://fnproject.io/tutorials/Flow102/)

The functions invoked by the flow can be written in any language that you choose.

Since the flow function is written in code, you could put embed other operations into it, but the best advice is to take a "separation of concerns" approach and to use the flow purely for orchestration, and deliver the business functionality via the invoked functions.

One way of thinking about this is to use an analogy with theatre.  The **functions** required for the play might be things like `add_character`, `move_character` etc. while the **flow** that combines these functions to deliver the play is the **script**.

You can see how this approach could be used to adapt Shakespeare's comedy ___As You Like It___ for the serverless world [here](https://bitbucket.org/ewan_slater/comedy/src).

The flow (or script) is implemented by the function `as_you_like_it`.  If you inspect the source code for the class `AsYouLikeIt.java` you will see that the flow (or *script*) adds various characters to the stage, causes them to variously fall in love, wrestle, run away, and disguise themselves, causing much confusion to the characters and (hopefully) amusement for the audience.

It finishes with all the main characters getting married (phew!).

One limitation that we currently have with Flow is that we need to invoke the functions by ID (rather than their name) so once the application has been deployed it is necessary to run a script (`self_configure.sh`) to configure a key - value lookup for each of the function IDs.

The `AsYouLikeIt` class then uses this to find the ID of the function it needs to invoke from the flow.

---

## <a name="saga"/> Implement a Saga with Fn and Flow

Sadly in the real world, things rarely run so smoothly.  Often we need to execute transactions which span multiple systems which we don't own or control.

The classic approach to dealing with with such distributed transactions was to use a two - phase commit (2PC) protocol and some kind of Transaction Manager (e.g. Tuxedo) to coordinate commit and rollback operations.

However, this approach typically impacts performance, does not scale well and requires the participating systems too support the protocol.

In many cases today we are likely to be dealing with systems which we do not control and which do not support a two phase commit protocol.

The **[Saga Pattern](https://microservices.io/patterns/data/saga.html)** describes how to implement reliable _business transactions_ across multiple systems in situations where a 2PC approach is not appropriate.

The overall business transaction is the "saga".  The saga is implemented as a set of individual (system - level) transactions.  If any of these steps fail then rather than _rolling back_ the transactions in the other systems (which effectively means they have never happened), a _compensating transaction_ is applied.

So for example if part of the saga involves debiting a customer's bank account, the compensating transaction will credit the bank account by the same amount.  The customer will see both the credit and the debit transactions on their bank statement.  This is different to a _rollback_ where the customer would not see either transaction.

Flow allows us to implement the saga pattern using Fn functions:  https://github.com/fnproject/tutorials/tree/master/FlowSaga
