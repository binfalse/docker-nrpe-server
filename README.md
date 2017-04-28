# Dockerized NRPE server


## Usage





## Build Image

You can of course build the image yourself instead of taking the pre-built versions from the Docker Hub.

As all versions use the [`src/nrpe-runner` tool](src/nrpe-runner) the root of this repository needs to be the build context for Docker.
The Dockerfile then needs to be specified manually using the `-f` flag.
That means, if you want to build the plain version you would call something like this:

    docker build -t nrpe-server:plain -f nrpe-plain/Dockerfile .


