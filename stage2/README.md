# microservice-tutorial: Stage 2

**Note**: Working code examples for this stage can be found in the [`stage2-sol`](https://github.com/gapoorva/microservice-tutorial/stage2-sol) folder.

In this stage, we'll start talking about [docker](https://www.docker.com), a popular linux container system that can run on most operating systems, and why it helps us implement microservices. We'll be adding docker to our local system, "pulling down" some images and running containers to get an understanding for what docker can do.

## What is Docker?

Docker calls itself a "container platform". It allows you to run [docker containers](https://www.docker.com/resources/what-container) on what's called the [docker engine](https://www.docker.com/products/docker-engine). Before understanding all these terms, let's briefly explore the core problem docker is trying to solve.

When running an application that serves real users, it's important to make sure the application works as intended. Often, the environment an application is developed in is different from the one it is run in. For example, most cloud applications run on linux servers, since these environments have strong software engineering communities and tools built on top of them. However, a developer may prefer to use a Windows or MacOS operating system to develop his or her software.

Colloquially, this is known as the "works on my machine" problem. Even if you develop an application on the same operating system as it runs on in "production", you can still run into problems if all the depencies of an application, like special files or SSL certificates, or networking setups are different.

Historically, software engineers have solved this problem by using virtual machines. In a virtual machine, an entire operating system and file system is emulated on the host computer. Virtual machines can still be expensive to run, since your application runs on several layers of abstraction. If virtual machine stops working, it's expensive to create a new one, since the machine images are huge. Finally, every VM needs to be setup independently - you can't send a VM to your teammate and have them run it without some required setup.

Docker aims to solve these problems using docker **images** and docker **containers**. Docker images are *composable descriptions* of a file system and system tools/libraries needed to run you application. You can define this file system with code, like you would any other program. Docker containers are *instances* of the images you define, and like virtual machines can be started or stopped. Unlike VMs however, containers only include what is needed to run your app, so they are very small and less resource intensive. You can also save docker images and store them in a repository so that other developers can retrieve them and run them in the exact same environment. In fact, docker images can run on any machine that has the docker engine installed - including the linux server that hosts your application to users!

## Installing Docker

Let's get started using docker. Below are instructions for installing docker on 3 popular operating systems. For more operating systems, see the [Docker installation guide](https://docs.docker.com/v17.12/install/).

* [MacOS](https://docs.docker.com/v17.12/docker-for-mac/install/)
* [Windows 10](https://docs.docker.com/v17.12/docker-for-windows/install/)
* [Ubuntu](https://docs.docker.com/v17.12/install/linux/docker-ce/ubuntu/#os-requirements)

We won't need to install other softwares for this tutorial - docker will allow us to use any tool we want! From this point on, since we'll be running utilities in the same environment, we won't need to make special exceptions for operating systems anymore :)

## Pulling your first Docker image

Let's start with the `hello-world` image. This is an image made by the creators of docker that will run a simple binary, print some output, and then exit.

Ensure docker is available from your command line:

```
$ docker -v
Docker version 18.06.0-ce, build 0ffa825
```

Then, let's directly run:

```
$ docker run hello-world
```

This will cause the docker engine to check locally for the `hello-world` **image**. Since it doesn't find it locally, it pulls from the docker registry - which is a public, shared registry. Once the image is downloaded, it will attempt to run it by creating a new **container**. That container is then executed, and the output is sent to the terminal. After the process ends, the container stops and becomes dormant.

To see your dormant container run:

```
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
71f02a8bbf42        hello-world         "/hello"            3 minutes ago       Exited (0) 3 minutes ago                       stoic_stallman
```

This will show you containers on your system. The `-a` flag makes docker show you both stopped *and* running containers. Ommitting it will show you only running containers. For this reason, think of containers like individual processes on your machine like your web browser or a text editor. Once they stop, the life of the process is over.

As you continue using docker you will accumulate stopped containers and unused images. It's a good idea so prune your images and containers periodically. Just run the following commands (agreeing to any questions):

```
$ docker container prune
$ docker image prune
```

## Running a docker image interactively

We can run docker images interactively by specifying a few special arguments to docker run, which will set up some configurations and make it appear as if we have `ssh`'d inside a VM.

Quick overview of options:

* `--rm` Removes the container after running it, so you don't collect extra containers
* `-it` Attaches an interactive TTY to the container so that you can send and receive signals to the container

Try it with an ubuntu image:

```
$ docker run --rm -it ubuntu
```

Docker will first pull the image `ubuntu` if it's unable to find it locally. Then it will start up the container and then execute the default entrypoint for this image, which is `bash`.

### Other useful options

You can pass environment variables using `-e`:

```
$ docker run --rm -it -e "FOO=BAR" ubuntu
root # echo $FOO
BAR
```

You can change the command that is run when the container is started:

```
$ docker run --rm -it ubuntu ls
bin   dev  home  lib64	mnt  proc  run	 srv  tmp  var
boot  etc  lib	 media	opt  root  sbin  sys  usr
```

You can change the working directory:

```
$ docker run --rm -it -w "/home" ubuntu pwd
/home
```

## The Dockerfile

Until now, we've worked with two different images. One was the `hello-world` image, and the other was the `ubuntu` image. Docker gives us the ability to define our own images that have all the pieces we need to run our apps.

We can use a `Dockerfile` to define an image. This allows us to set up a lot of the environment for our app. This can include:

* Installing binaries such as node, python, php, etc...
* Adding in source code files
* Adding in configuration files
* Setting up a working directory
* Changing the user to boot the machine with
* Setting up port forwarding
* Setting up a default "entrypoint"
* Setting up a default command on the entrypoint.

We can use this to build our image and ship it like an "executable". As an example, let's use this small bash script and dockerize it.

**replace.sh**

```bash
#!/bin/bash

echo $1 | sed "s/$2/$3/g"
```

This is a small (and trivial) wrapper around the sed command that accepts content as `$1`, a search regex as `$2` and a replacement string as `$3`. Below is example usage:

```
$ ./replace.sh "my cat ate a bat and then a hat" "[c|b|h]at" warble
my warble ate a warble and then a warble
```

We can use a dockerfile to build a docker image whose sole purpose will be to run this command, in a consistent, ubuntu environment. No matter which host OS this image is run on, thanks to docker it will behave the same.

First, let's create a dockerfile:

**Dockerfile**

```Dockerfile
# FROM specifies which image to inherit from - this helps reduce the number of things we'd need to potentially redefine/install in order to have a working ubuntu environment.
FROM ubuntu 

# COPY command copies files in from the "build context" to the image. This would be used to copy in source files and configuration files.
#
# Here, we're copying replace.sh from the "build context" directory to the "/home" directory within the image.
COPY ./replace.sh /home

# WORKDIR changes which directory is considered the "working directory" inside the image. The last setting will then be the default location in the file system when the image is run.
WORKDIR /home

# RUN executes a command in the shell. This is the way we can run executables to have side effects like installing dependencies or compiling code. The command is run within the last WORKDIR setting. Here, we're making sure that `replace.sh` is executable.
RUN chmod +x replace.sh

# ENTRYPOINT sets up the executable that should be run inside the docker container. This can't be overriden by the user when running `docker run`, essentially making this command the single focus of this image.
ENTRYPOINT ["/bin/bash", "/home/replace.sh"]

# CMD sets what the default command arguments should be, and is passed to the ENTRYPOINT as a set of default arguments when none is provided by the user. This is why CMD is an array. In our case, there isn't a good default for the replace command so we will omit this from the Dockerfile.
#
# If we were to uncomment the line below, by default the container would print "jello" and then exit when run without any parameters.
# CMD ["hello", "h", "j"]
```

This Dockerfile _codifies_ the environment that we want to set up, so we can be sure that our image works the same way every single time. We can use the Dockerfile and the `docker build` command to build our new image:

```
$ docker build -t replace-executable:1.0.0 -f ./Dockerfile .
# a bunch of output ....
Successfully built 609bf6c49a23
Successfully tagged replace-executable:1.0.0
```

**Explanation of options**:

* `-t replace-executable:1.0.0` aliases our image as "replace-executable" in addition to the SHA that docker generates. In the above example, that's `609bf6c49a23`. We can add an additional (optional) qualifier to the name as a "tag" - in practice this is often used to specifiy a version and when omitted, docker assumes that tag should be `latest`.
* `-f ./Dockerfile` specifies a path to the Dockerfile that `docker build` should use. When the path is `./*.Dockerfile`, you can actually omit the `-f` option, `docker` automatically searches for a `*.Dockerfile` in your current working directory.
* Notice at the end of the `docker build` command is a `.`. Very subtle, the dot is important because that argument is specifying the **build context**. This is how `docker` knows where to grab your source code from. You can of course run the `docker build` command using a different build context than your current working directory.

**Explanation of output**

What's happened here is that `docker` executed the commands in our Dockerfile in sequence to arrive at our new image.

As it builds, it spits out SHA tags like `---> cd6d8154f1e1`. These are the SHA identifiers of "layers". Each command in the Dockerfile causes `docker` to build a new "layer" using the previous layer as a starting point and then adding the changes from the command. The resulting filesystem is hashed and the layer is given that ID.

This way, when we go to rebuild the image after changing some source code, we only have to re-run the steps starting with the step that copies in source code. Layers before then don't have to change and therefore don't have to be re-computed. This saves a bunch of time when developing on docker.

If any command in the Dockerfile results in an error, `docker build` will spit out an error and stop building. This way `docker build` is almost like a compiler for your environment.

We now have our first custom docker image! Hooray! We can confirm that our docker image really does exist by running `docker images`:

```
$ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
replace-executable   1.0.0               96b92c6cddc6        2 minutes ago      84.1MB
# Potentially more images also listed ...
```

Now, just like the `hello-world` and `ubuntu` images, we can run our `replace-executable` image as well!

```
$ docker run --rm replace-executable "my cat ate a bat and then a hat" "[c|b|h]at" warble
my warble ate a warble and then a warble
```

What we've done is made an executable out of our docker image - one that is very good at replacing text in strings. There's a slight downside in that it takes a bit more typing, but it's nothing a small bash script or a bash `alias` couldn't cure :). As an added benefit, you can now run your image anywhere, including a new terminal window, different working directory or as we'll see below on you friend's computer!

## Sharing your images

Going back to the beginning of this stage, our goal with learning `docker` was to solve the "works on my machine" problem. So it's important to learn how you can share and distribute your docker images. With this ability you can send your docker images to your teammates, run it on a raspberry pie, or transfer it to your cloud machine so that you can expose your app to the wider internet audience.

### Docker Hub method

One way you can share your docker images is by using a **docker registry**. This is basically a remote server that can manage image uploads and downloads so that anyone with access to the internet (and your images) can publish new versions of an image and pull latest versions. It works a lot like cloud SCM services like Github or BitBucket.

You could host your own private registry - and most serous software organizations will consider doing this to save money and secure their technology - but for small and open source projects, the public docker registry called "docker hub" should work great.

First head over to [hub.docker.com](https://hub.docker.com) to create an account with docker. Then, login to your local docker agent:

```
$ docker login -u <username>
```

You will be prompted for your password, after which you should be authenticated to your docker agent. You can now push images to the docker hub under your account.

To publish your docker image, you need namespace your image under your username. To do this, let's build another version of `replace-executable` but under your username namespace.

```
$ docker build -t "<your_username>/replace-executable" -f ./Dockerfile .
```

You can now publish the docker file to your docker hub account:

```
$ docker push <your_username>/replace-executable
```

After some pushing... tada! You have published your first image to the docker hub. You can use the UI on [hub.docker.com](https://hub.docker.com) to manage access control rights and collaborators who can push to your docker repo using the same name.

Now, your friend can run this image locally with just one command (as long as they too have docker installed):

```
$ docker run --rm <your_username>/replace-executable "my cat ate a bat and then a hat" "[c|b|h]at" warble
my warble ate a warble and then a warble
```

It doesn't matter which environment you developed your app on, it doesn't matter which environment you need to run your app in. As long as both have `docker`, you can run any kind of tech stack. With nothing but a favorite text editor and `docker` you can build most every software application you can think of.

## Further Reading

For our tutorial, this was a fairly thorough overview of docker. There are plenty of tools we didn't cover, and the best place to read about them is the [docs.docker.com documentation](https://docs.docker.com).