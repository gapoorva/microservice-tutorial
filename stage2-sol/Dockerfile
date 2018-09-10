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