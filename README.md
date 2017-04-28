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

    $ docker run -d -p 5666 binfalse/nrpe-server:full
    NRPE v3.0.1


This will run the NRPE server and listen to commands on port 5666 of your host machine.
From any host (e.g. `localhost`) that has the NRPE plugin installed you should be able to communicate with that server:

    ./check_nrpe -H localhost -n

The image comes with three volumes:

* `/etc/nagios/nrpe.d/` the place to mount your custom configurations for the server, including all necessary commands etc
* `/etc/nagios/certs/` if you enable SSL the NRPE by default expects the following files to be mounted in this directory:
  * `chain.txt` your CA-certificates including all chain certificates
  * `cert.cer` the SSL certificate
  * `key.pem` the SSL key
* `/usr/lib/nagios/extra/` optional plugins that you need to add to the container


### Environment variables

For convenience, some configurations of the NRPE server can be done through environment variables:

* `ALLOWEDHOSTS` may carry a list of IP addresses (comma separated) that are allowed to communicate to the NRPE server, see also . By default everyone is allowed to send commands. Setting `ALLOWEDHOSTS` is therefore highly recommended to enhance the security. Example: `ALLOWEDHOSTS=1.2.3.4,192.168.0.1/24`
* `PORT` can be used to have the listen to a specific port. If `PORT` is left empty it defaults to `5666`. For example, `PORT=8008` will tell the server to listen to port `8080` in the container.
* `SSL` is a boolean option. If set to `yes` the NRPE server will require encrypted communication. In that case, you also need to configure certificates -> [see the section on SSL below](#ssl).

A call may thus look like this:

    docker run -it --rm -p 4444:1234 -e PORT=1234 -e ALLOWEDHOSTS=172.17.0.1 -e SSL=no binfalse/nrpe-server:full

Here, the machine `172.17.0.1` (`ALLOWEDHOSTS`) will be able to talk unencrypted (`SSL=no`) to the NRPE server on port `4444` (`PORT`) of its host.

### Configure monitoring

Monitoring setup needs to be mounted to `/etc/nagios/nrpe.d/`, every file that ends in `.cfg` will be evaluated by the NRPE server.

Let's assume you want to monitor the webserver of `binfalse.de`. Then you would create a file `/PATH/binfalse.cfg` containing

    command[check_binfalse]=/usr/lib/nagios/plugins/check_http -H binfalse.de

(Here it uses the `check_http` tool that only comes with the `nrpe-server:full` version.)
You then need to mount that file into the container:

    docker run -it --rm -p 5666 -v /PATH/binfalse.cfg:/etc/nagios/nrpe.d/binfalse.cfg binfalse/nrpe-server:full

Done. You should now be able to execute that command from any host through the `check_nrpe` tool, eg:

    $ ./check_nrpe -H localhost -n -c check_binfalse
    HTTP OK: HTTP/1.1 301 Moved Permanently - 375 bytes in 0.044 second response time |time=0.043561s;;;0.000000;10.000000 size=375B;;;0



### Add own monitoring plugins

If you're missing a monitoring plugin in the Docker image you can just mount it to `/usr/lib/nagios/extra/` directory and use it as usual.
Let's say you wrote the script `/PATH/scripts/check_something`, then you need to create a monitoring config file `/PATH/conf/mymonitor.cfg` containing the command:

    command[check_something]=/usr/lib/nagios/extra/check_something -ARG1 -argument2 -x Arg3 ...

Then mount both to the appropriate locations in the container:

    docker run -it --rm -p 5666 -v /PATH/scripts/:/usr/lib/nagios/extra/ /PATH/conf/:/etc/nagios/nrpe.d/ binfalse/nrpe-server:full

And that's it. You should now be able to execute that command by sending `-c check_something` through the `check_nrpe` plugin.


### SSL


## Docker-Compose



## Build Image

You can of course build the image yourself instead of taking the pre-built versions from the Docker Hub.

As all versions use the [`src/nrpe-runner` tool](src/nrpe-runner) the root of this repository needs to be the build context for Docker.
The Dockerfile then needs to be specified manually using the `-f` flag.
That means, if you want to build the plain version you would call something like this:

    docker build -t nrpe-server:plain -f nrpe-plain/Dockerfile .


## LICENSE

	docker-nrpe-server -  Dockerized version of Nagios' NRPE server 
	Copyright (C) 2017 Martin Scharm <https://binfalse.de/contact/>

	This program is free software: you can redistribute it and/or modify
	it under the terms of the GNU General Public License as published by
	the Free Software Foundation, either version 3 of the License, or
	(at your option) any later version.

	This program is distributed in the hope that it will be useful,
	but WITHOUT ANY WARRANTY; without even the implied warranty of
	MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
	GNU General Public License for more details.

	You should have received a copy of the GNU General Public License
	along with this program.  If not, see <http://www.gnu.org/licenses/>.
