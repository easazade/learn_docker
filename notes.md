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
EXPOSE: Declares the port(s) on which the container will listen for connections. This is more of a documentation hint than a rule, as it doesn’t actually expose the port to the host.
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
