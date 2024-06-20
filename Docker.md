# Docker

## what it is and why use it?

To Make our life easier to install and run software on any computer / cloud / server

for ex. to install redis

> \_ docker run -it redis

It is a platform or an ecosystem around Containers

A container is an instance of a docker image that runs a program (single file with all the deps and config to run a program)

Docker Client (cli) and Docker Server (daemon)
the cli is a tool to help us interact with the docker server (the tool responsible for creating image, running containers...)

## installing on windows

Require a docker signup 
(fffjacquier)

and install >_ wsl --install 
that will install windows subsystem linux

then install docker desktop
restart
in wsl create login pass for ubuntu
(fjacquier)

then all docker commands should be run from the wsl console
>_ docker login
will ask for username and password (docker)
When Login Succeeded message, you are set

## Using the docker client

docker run hello-world

On the computer: docker client communicates with docker server to create an image hello-world in image cache.
Starting, there is no image cache, so the server picks on docker hub to search for hello-world

on the docker hub: hello-world
it's downloaded and put in the Image Cache on the computer

the 2nd time you run the command, you'll not see the unable to find the image hello-world you saw on the first time

## So what is a container?

Most op. system have a kernel, a software process that have access to your programs running and the hardware of the computer CPU, Memory, Hard Disk
The Kernel is interacting with programs with system Calls to interact with piece of hardware

The kernel checks which process is making the system call and redirect it to segment of HD for the same process

This is namespacing (isolating resources per process) for Processes, Users, HD, Network, Hostnames, inter process comm.

Control groups (cgroups): limit amount of resources used per process is for Memory, CPU usage, HD I/O, Network Bandwidth

Namespacing and cgroups are linux specific features

A container is the entire vertical between the process and the namespacing / cgroup of the process on the HD

Image = File system snapshot + startup command (chrome python | run chrome)
Container = running process (fs snapshot) -> Kernel -> ram | network | CPU
and only the Running processes space on HD

## How is Docker running on your computer?

you install a Linux virtual machine with Docker. It has a linux kernel.

## Docker commands

**create and run a container from an image:**
> docker run hello-world

> docker run <image name> command 
command is an override
(docker run busybox echo hi there)
-> will echo 'by there in the terminal)

if you type
> docker run busybox ls
you'll see the linux fs folders of the container 
if we rerun docker run hell-world ls or echo you'll get errors

with busybox there are in the image the ls or echo program, but they do not exist on the hello-world image container

## Listing running containers

docker ps

if we have done on a tab
> docker run busybox ping google.com
and run `> docker ps` on another tab
we'll see a table like that:
CONTAINER ID   IMAGE     COMMAND             CREATED          STATUS          PORTS     NAMES
b1e6e33a2a7b   busybox   "ping google.com"   37 seconds ago   Up 36 seconds             naughty_hermann

docker ps --all -> will show all the container started

Useful to get the container ID

## Container lifecycle

docker run = docker create + docker start
_create_ will take the fs snapshot in the image and put in in the Container (segment on hard drive)
_start_ is about executing the startup command

docker create hello-world -> will return a long-id
docker start -a long-id -> will start running the command
-a : *attach* to the container and watch output and give it back in the terminal console

# restart stopped containers

when a container is exited you can execute
docker start containerid or with -a
the running process inside the container currently will be the "echo hi there" override but we cannot replace it

# removing stopped containers

> docker system prune
it will remove stopped containers, networks not used, dangling images, unused build cache


## Retrieving output logs

> docker logs containerid

copy paste in wsl:
activate the checkbox in the properties

here you are just getting a record from the logs that have been emitted by the container, you are not starting it

## Stopping containers

> docker create busybox ping google.com
> docker start containerid
> docker logs containerid
> docker ps
> docker stop (or kill) containerId
-> stop sends a SIGTERM (terminate signal) to the running process in the container giving 10 secs to the container before killing it (chance to stop graceful)
 kill sends a SIGKILL signal meaning stop immediately

## multi command containers

> docker run redis -> redis-server will start
but you cannot run redis-cli outside the container with redis-server

so how to do the two commands in the container?
Use the exec command with -it
> docker exec -it <containerid> command
docker exec -it 1aa89a684b7c redis-cli
the -it flag is for 'input text'

in linux env. processes run with 3 channels (STDIN STDOUT STDERR)
the -it is 2 separate flags -i and -t
-i: when we execute the new command we want to attach what we type to the process
-t: make sure that all the text is nicely formatted on the screen 
if you omit the -t you'll see no prompt, no autocomplete, ...

## Getting a command prompt shell in a container

> docker exec -it 1aa89a684b7c sh  
we'll see a # which is a shell

if ctrl+c does not work try ctrl+d

sh is a command processor, a shell that can be executed 
zsh or bash or powershell too

or you can also do:
docker run -it busybox sh

## Container isolation

between two containers, they do not share their data or file system

## Creating docker image

Dockerfile: config to define how our container behave 
Dockerfile -> docker client -> docker server -> Usable Image!

Flow:
specify a base image
run some commands to install add. programs
specify a command to start the program

Exercice: Create an image that runs redis-server
> mkdir redis-image 
> cd redis-image 
> code .

Create a Dockerfile (no extension, uppercase)
````config
# Use an existing base docker image
FROM alpine

# Download and install dependencies
RUN apk add --update redis

# Start
CMD ["redis-server"]
````

then 
> docker build .
will exec the Dockerfile and give an id at the end
> docker run the-id

FROM
RUN
CMD:
are instructions telling Docker Server what to do

after the instructions come the argument to the instruction

## what is a Base image?

Writing a dockerfile is like being given a computer without OS and being told to install google chrome on it
Install an operating sytem
Startup your defautl browser
Navigate to chrome.google.com
Download Installer
Open Explorer
Exec chrome_installer.exe
Execute chrome.exe

The alpine docker image: it comes with a preinstalled set of programs that can be useful for redis!

apk: apache package kit, to download and install redis

## The Build Process  

Sending build context
Steps 1/3 (from alpine)
alpine image with FS Snapshot and Startup command (??)

Step 2/3 (run apk)
create a temp container from the alpine image
then the running process is apk add --update redis inside the container
so the process is exec (install et run redis)
and the temp container is stopped and the new fs snpashot is saved as new image that has a installed version of redis


Step 3/3 (cmd redis-server)
the new image creates a container with the FS Snapshot with redis
and adds the redis-server as running process
and take a new snpachot of its FS and running primary command as a new image and remove the previous image

Every time you make a change on the Dockerfile, docker will use the cached version
unless you change the order of the steps, so add changes after what is already there.

**How to tag a docker image in order to run docker run myimage instead of the image id?**
> docker build -t fffjacquier/redis:latest . 

your dockerid / <projectname> : version
the dot at the end is important: it's where the files are
the tag is the version, the rest is the folder
community images does not have docker id of course

then
> docker run fffdocker jacquier/redis

**Manual installation with docker commit:**
docker commit -c 'CMD ["redis-server"]' CONTAINERID

If you are a Windows user you may get an error like "/bin/sh: [redis-server]: not found" or "No Such Container":
docker commit -c "CMD 'redis-server'" CONTAINERID

docker run -it alpine sh
/ # apk add --update redis
/ #

open a second terminal window:
> docker ps // to get container id
> docker commit -c 'CMD ["redis-server"]' CONTAINERID
> docker run newcontainerid

## Build a nodejs web server on docker

> docker build .

common errors: npm not found
use: FROM node:14-alpine -> specify a version

or     
FROM node:alpine
WORKDIR /usr/app
if the folder does not exist it will be created 

Then error: no such file package.json
All your files are not available unless you specify it in the dockerfile
We have to copy all our files
COPY ./ ./ -> path to folder from your local file system relative to build context (our . line 278)
           -> place to copy stuff inside the container (here WORKDIR)
  
docker build -t fffjacquier/simpleweb .   

**stop wsl: ctrl+d or ctrl+c or ctrl+z**

Next issue: even when we get the server log on terminal, we cannot access the localhost:8080 in a browser!
The content is not routed to the container by default (for incoming requests)
To do that, we have to setup an explicit port mapping:
  forward the port to the container: 

  > docker run -p 8080:8080 containerid
  the first 8080 is the route incoming request to this port on localhost
  the second 8080 is the port inside the container
  ( docker run -p 8080:8080 fffjacquier/simpleweb  )
  on pourrait faire 5000:8080 et appeler localhost:5000 dans le browser (8080 ds nodejs)


> docker run -it fffjacquier/simpleweb sh
or on another tab terminal
> docker ps // to get the containerid
> docker exec -it containerid sh

## Unnecessary rebuilds

if we make a change on our nodejs app, we do not see the change on browser
changes are not reflected in our container
We have to rebuild the container for that
All steps are replayed even if we should only rerun the COPY command and not reinstall all dependencies again.
How could we do that?

split the copy in order to not invalidate the cache on next builds:
COPY ./package.json ./
RUN npm install
COPY ./ ./

Common other commands on 2nd terminal
> docker logs <containerid>
> docker exec -it <containerid> sh
