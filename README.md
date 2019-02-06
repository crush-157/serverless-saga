# Writing a Serverless Saga - in code

In this workshop, we're going to create a serverless implementation of the [saga pattern](https://microservices.io/patterns/data/saga.html).

We're going to build this using [Fn](https://fnproject.io/), an open source, container - native, serverless platform.

## Agenda
1.  [Introduction to Fn](#introduction)
2.  Install Fn
3.  Create your first function
4.  Group functions together in apps
5.  Orchestration with Flow
6.  Implement a Saga with Fn and Flow

## <a name="introduction"/> Introduction to Fn

## Installing Fn (5 - 10 mins)
Initially you'll need to install the Fn environment.

We've provided you with the login details of a Linux instance running in the Oracle Cloud.  This has docker installed and running, and also a networking configuration with the right cloud environment external rules.

Fn will run anywhere that Docker runs including cloud(s), on - premises servers and your own laptop.

To download and install the Fn CLI, follow these [instructions](http://fnproject.io/tutorials/install).

Then use that to start the Fn server (`fn start`) and test it.

You will need to use two ssh sessions here, one to run the server, the other to interact with it.

If you prefer you can run the server in the background by using the -d flag (`fn start -d`).

In this case you won't see the startup text, but will see the docker container id.

If having started the server you decide to move it to the background you can stop the server with Ctrl-C or run the command `$fn stop` in another window, then re-start it with `fn start -d` (If you restart it it doesn't need to download the fn server docker images, so it's much faster on subsequent starts)

If you do start the server in the background be sure to have it running when you start configuring the context.

As we're using the script to setup fn you can skip the Fn Manual Install section

## Introduction exercises
Fn supports many languages, and you can even run a shell script in a plain docker container as a function if you like.

But it's fun to do some simple introductory type functions.

These are available in several languages and will take you between 10 and 20 mins depending on the language.

Because we're using Fn however it will automatically do all of the work needed to provide the compile - time and run - time environments.  So all you need to is tell Fn what language your function is to be written in and create the code.

The following languages have built in support in Fn, follow the links to go to a tutorial in that language.


- [Go](http://fnproject.io/tutorials/Introduction/)
- [Java](http://fnproject.io/tutorials/JavaFDKIntroduction/)
- [Node.js](http://fnproject.io/tutorials/node/intro/)
- [Python](http://fnproject.io/tutorials/python/intro/)
- [Ruby](http://fnproject.io/tutorials/ruby/intro/)

If you like you can do multiple introductory tutorials, Fn does all the work needed to support multiple languages, and because under the covers it uses docker containers there's no need to worry about them conflicting.


## Beyond Hello World
Once you've got your Hello World! function running let's see how Fn can group multiple functions together to create an application (OK, this is a *very* simple application, it just responds to requests on two different url's).

One interesting point here is that the functions are in multiple languages, but because of the runtime compartmentalisation the Fn server provides they still run happily together as a single app.

If you want to explore more have a look at the various [FDK's (Function Developer Kit)](https://github.com/fnproject).

There is a range of examples, for example [generating QR codes](https://github.com/fnproject/fdk-java/tree/master/examples/qr-code).