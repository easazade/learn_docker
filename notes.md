###################################################################################################
the `ENV` instruction allows us to define environment variables inside the container.
if our app is using environment variables, then when we put the app inside the container we need to have 
those variables defined inside the container environment. and that can be done using ENV

```docker
ENV PORT=8080
ENV MEOW_API_KEY="your-meow-api-key"
```

alternatively environment variable can be overridden when running an image:
`docker run -e PORT=9090 -e MEOW_API_KEY="new-key" <image-name>`
###################################################################################################
Differences between ENTRYPOINT and CMD are that:
with ENTRYPOINT we set the default command or app that needs to run and that should not be changed
EVEN though the CMD can do the same but CMD can be overridden.

SO a typical use case could be like below:

```docker
ENTRYPOINT ["dart", "bin/main.dart"]
CMD ["--no-debug-logs", "-p", "8080"]
```

where cmd is used for arguments and it can be overridden, when running docker image.
`docker run <image-name> --verbose -p 8090`
###################################################################################################
To remove a container run `docker rm <container-id>`
###################################################################################################
To start/stop/attach a container run
NOTE: you can check all available containers with : docker ps -a 

`docker start <container-name>`
`docker start <container-id>`
`docker attach <container-id>` # attaches to already running container
`docker stop <container-id>`
`docker start -a <container-id>` # starts a container and attaches to it
`docker start -ai <container-id>` # starts a container and attaches to it in interactive mode
###################################################################################################
when running a container `-d` flag runs it in detached mode
###################################################################################################
If you run the docker image and the server/app not working correctly. you should:
1- check the logs. `docker logs <container-id>`
2- try to see if host is 0.0.0.0 and not localhost and port mapping is done by running `docker ps`
3- make sure server is also running on correct ip and port.
4- try to check for connectivity from inside the docker container by running 
`docker run -it --rm -p 7899:7899 dart-server /bin/sh`
5- Another things that helps for testing is overriding CMD instruction when running a docker image in a way that helps us debugging
eg: `docker run <image-name> <any-command>` like docker run dart-server echo "MEOW"
or we can pass additional args to CMD command 
eg: `docker run dart-server:v1 --verbose` in this example the `--verbose` will be added as extra argument to CMD instruction defined.
NOTE: that we cannot do any of this for a container. only for images.
###################################################################################################
Always Ensure the Dart Server is Listening on 0.0.0.0
In Docker, by default, the containerized application should listen on all network interfaces (0.0.0.0) 
to be accessible from outside the container. If the Dart server is only listening on 
localhost (127.0.0.1), Docker won't expose it.

Check your Dart server code (e.g., server.dart) and look for where it binds to an address.
It should bind to 0.0.0.0 instead of 127.0.0.1.
###################################################################################################
docker has an interactive mode where for example you can run an ubuntu image and the execute 
shell scripts on it from terminal. eg: 
docker run -it ubuntu

the good thing about interactive mode is that you can access the image and inspect it.
###################################################################################################
A container is like a process to see running containers call: docker ps
to see all containers run : docker ps -a
###################################################################################################
You can pull any image from docker-hub and then just run it:
docker pull ubuntu
docker run ubuntu
###################################################################################################
The alpine linux is a very small distribution of linux. images built on top of it will be around 100 MB
NOTE: transfering large images to remote registries is hard. try to keep images small
eg:
```docker
FROM node:alpine
COPY . app/
WORKDIR app/
CMD node app.js
```
###################################################################################################
To build an image from a Dockerfile run `docker build -t your-tag`
The -t is used to identify the image by name and also add tag to it
Remember that by default docker adds a tag `latest` on the image
each created image also has an `IMAGE ID`
###################################################################################################
To see all built docker images run
docker images
or
docker image ls
###################################################################################################
if you call `docker version` command it shows info for a client and a server
the client is the cli that we use the server it is connected to is the docker engine that runs, builds
the images. we can make the client (cli) to connect to any other server 
like a remote one (instead of desktop one installed on our machine)
###################################################################################################
EXPOSE instruction:
EXPOSE is primarily a way to inform others that a container listens on specific ports. For example,
EXPOSE 8080 suggests that the application in the container will be accessible on port 8080.

EXPOSE alone does not make the port accessible outside the container. To make the port accessible, 
you need to map it when running the container with the -p or --publish option in the docker run command.

in a docker file like this:
```docker
EXPOSE 8080
CMD ["/app/bin/server"]
```
We should run `docker run -p 8080:8080` for port mapping
to verify port mapping check running processes by `docker ps`

###################################################################################################
When setting the stage in example like this `FROM dart:stable AS build`
the word after image name separated by : is the version of the base image.
dart:beta
dart:stable
dart:3.5.0
###################################################################################################
in `FROM dart:stable AS build` we are setting dart image with version stable as base image with the 
stage name of `build`
###################################################################################################
Each time we are calling FROM instruction we are setting a new base image for a new stage.
Then we build our own image layer by layer on top of the base image. each stage is isolated from other stages
but we can access them or files inside them by their stage name
###################################################################################################
the files listed in .dockerignore will be hidden/ignored by docker instructions. eg: if a file is ignored
COPY command does not work on it. 
###################################################################################################
`From scratch` is a minimal base linux container.   
No operating system files or libraries (no /bin, /usr, or any system utilities).
No shell (like bash or sh), so any commands requiring a shell won't work.
No standard utilities (e.g., no ls, cat, or ping).
###################################################################################################
A list of all Docker Instructions:

FROM: Sets the base image for subsequent instructions. This is usually the first command in a Dockerfile.
WORKDIR: Sets the working directory inside the container for any following instructions like RUN, CMD, ENTRYPOINT, etc.
COPY: Copies files and directories from the host system into the container's filesystem.
RUN: Executes commands in a new layer on top of the current image. Commonly used for installing software packages.
CMD: Specifies the default command to run when the container starts. Can be overridden by passing a command when running the container.
ENTRYPOINT: Sets the executable that will run in the container. It works similarly to CMD but is not overridden as easily. Often used with CMD for flexibility.
ENV: Sets environment variables in the container. Useful for configuration values that may be needed at runtime.
EXPOSE: Declares the port(s) on which the container will listen for connections. This is more of a documentation hint than a rule, as it doesn't actually expose the port to the host.
ADD: Copies files/directories from the host, similar to COPY, but also has additional features (like unpacking local .tar files). However, COPY is preferred for clarity.
VOLUME: Creates a mount point with a specified path in the container and marks it as holding externally mounted volumes or persistent data.
USER: Sets the user for subsequent instructions and for running the container. This can enhance security by not running containers as root.
ARG: Defines variables that users can pass at build time. Useful for setting build-time variables that are not persisted in the image.
ONBUILD: Sets a trigger instruction to run when the image is used as the base for another image. This is useful in base images intended for other developers.
STOPSIGNAL: Sets the system call signal that will be sent to the container to exit. This is useful for managing how a container should gracefully shut down.
LABEL: Adds metadata to an image, like versioning or author details. Often used to describe the image.
HEALTHCHECK: Tests if the container is still functioning as expected. It can define a command to check the health of the container, retry intervals, and other parameters.
SHELL: Specifies the shell to use for RUN commands on Windows (e.g., cmd or powershell). On Linux, it can override the default /bin/sh.
###################################################################################################
```docker
# Use the Dart SDK for Windows as the build stage
FROM google/dart AS build

# Resolve app dependencies.
WORKDIR /app
COPY pubspec.* ./
RUN dart pub get

# Copy app source code (except anything in .dockerignore) and AOT compile app.
COPY . .
RUN dart compile exe bin/server.dart -o bin/server.exe

# Build minimal serving image for Windows from the pre-built AOT-runtime
FROM mcr.microsoft.com/windows/nanoserver:ltsc2022
WORKDIR /app/bin
COPY --from=build /app/bin/server.exe .

# Start server.
EXPOSE 8080
CMD ["C:\\app\\bin\\server.exe"]
```
