# Hello world in .NET on IR1101 IOx

This Git repo demonstrates procedure and Dockerfile to build an .NET application
running on IOx framework coming with Cisco Catalyst IR1101 Rugged series router.

Building process consists of following steps.

1. Build a Docker image for source of IOx app.This repo provides Dockerfile with
   multistage.  The Dockerfile builds .NET Hello World app, publishes it as
   single file Linux ARM64 executable, and builds a runtime docker image
   containing the executable for you.
1. Convert the runtime image to IOx application package.

## Prepare your environment

Before you start, you need following:

- Docker which can run ARM64 containers
- `ioxclient` tool from Cisco

### Docker supporting ARM64 executables

This repo is using ARM64 container for building .NET application.  This repo
intentionally avoid using cross compilation environment for simplicity.  You
have to run ARM64 container inn order to build Hello World app.  You can do it
on native ARM64 platform or you can do it on QEMU emulated Docker environment.
Recent Docker environment running allows you running ARM64 docker container
seamlessly on x64 platforms.  Please refer Docker documentations.

Emulated environment is really useful but be aware compilation is significantly
slower than native environment.

### `ioxclient`

`ioxclient` is a developer tool used for manipulating IOx environment.
It will be used for creating IOx application from a Docker image.
You can find `ioxclient` [here](https://developer.cisco.com/docs/iox/#downloads).

Consult to [Cisco DevNet site](https://developer.cisco.com/docs/iox/#what-is-ioxclient) for more detail.

## Build runtime Docker image

Build a Docker image for source of IOx app.
Navigate to `docker` directory.  Execute following command.

```shell-session
$ cd docker
$ docker build -t ir1101-iox-hello-dotnet .
```

Docker will once create an image for build then create another image for
runtime.  You will get a Docker image tagged with "ir1101-iox-hello-dotnet".
The image contains Hello World app and a shell script for keeping container
running.

## Convert Docker image to IOx app

Once a Docker image for runtime has been built, you can convert it to IOx
application package.

Navigate to the directory `app` and execute following command.

```shell-session
$ cd ../app
$ ioxclient docker package ir1101-iox-hello-dotnet .
```

You will get `packaget.tar` which is IOx application package installable to
IR1101.

## Verify the app

Install the IOx app to IR1101, activate, then start.

When the app is started, `loop.sh` is called.  This sample app doesn't start
.NET Hello World because it cannot keep the app container running.  Like Docker
container, IOx app container stops when its first process terminated.  If you
started Hello World as the first app process, it shows Hello World message then
immediately finishes.  As a result of this, the entire app container stops
immediately so you cannot verify if the Hello World is really working.

So you starts a small dummy process which only keeps running so that you can
keep the app container running until you stop it intentionally.  The `loop.sh`
does this.

While IOx app is running, you can access to console of the container from IR1101
IOS CLI prompt.

```shell-session
Router#app-hosting connect appid <app-ID> session
```
Once you get shell prompt inside of the app container, run `hello` command to
execute the Hello World.

```shell-session
# hello
Hello World!
```

You will see "Hello World!" on the screen.