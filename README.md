# Dockerized NRPE server

This repository compiles into a Docker image containing a Nagios-NRPE-Server.
[Documentation for the NRPE server configuration](http://nagios.sourceforge.net/docs/nrpe/NRPE.pdf) can be found on the Nagios website, see also the [GitHub project page of NRPE](https://github.com/NagiosEnterprises/nrpe).

## Versions / Tags

The image comes in two flavours:

* `binfalse/nrpe-server:plain` Plain NRPE server that just contains the Nagios NRPE server including some basic config. If you need any plugin you need to bring it yourself.
* `binfalse/nrpe-server:full` NRPE server shipping lot's of monitoring plugins in it's `/usr/lib/nagios/` directories. To see which plugins are included just run the following command:

    docker run -it --rm --entrypoint find binfalse/nrpe-server:full /usr/lib/nagios

In the following I will use `binfalse/nrpe-server:full` for explanation, but most of that also applies to the plain version


## Usage

Basically, you just need to run the following:

    docker run -d -p 5666 binfalse/nrpe-server:full

This will run the NRPE server and listen to commands on port 5666 of your host machine.
From any host (e.g. `localhost`) that has the NRPE plugin installed you should be able to communicate with that server:

    ./check_nrpe -H localhost -n


### Environment variables

For convenience, some configurations of the NRPE server can be done through environment variables:

* `ALLOWEDHOSTS` may carry a list of IP addresses (comma separated) that are allowed to communicate to the NRPE server, see also . By default everyone is allowed to send commands. Setting `ALLOWEDHOSTS` is therefore highly recommended to enhance the security. Example: `ALLOWEDHOSTS=1.2.3.4,192.168.0.1/24`
* `PORT` can be used to have the listen to a specific port. If `PORT` is left empty it defaults to `5666`. For example, `PORT=8008` will tell the server to listen to port `8080` in the container.
* `SSL` is a boolean option. If set to `yes` the NRPE server will require encrypted communication. In that case, you also need to configure certificates -> [see the section on SSL below](#SSL).

### Configure monitoring


### Add own monitoring plugins




### SSL


## Docker-Compose



## Build Image

You can of course build the image yourself instead of taking the pre-built versions from the Docker Hub.

As all versions use the [`src/nrpe-runner` tool](src/nrpe-runner) the root of this repository needs to be the build context for Docker.
The Dockerfile then needs to be specified manually using the `-f` flag.
That means, if you want to build the plain version you would call something like this:

    docker build -t nrpe-server:plain -f nrpe-plain/Dockerfile .


