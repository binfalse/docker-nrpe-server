# Dockerized NRPE server


## Docker build

as all versions use the (`src/nrpe-runner`)[src/nrpe-runner] you need to use the root of this repository as the build context and specify the Dockerfile manually.
That means, if you want to build the plain version you would call something like this:

    docker build -t nrpe-server:plain -f nrpe-plain/Dockerfile .


