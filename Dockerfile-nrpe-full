FROM debian:testing-slim
MAINTAINER martin scharm <https://binfalse.de>

# nrpe server cannot log to std::out, so we need syslog2stdout (i provide a package at apt.binfalse.de)
RUN apt-get update \
    && apt-get install -y --no-install-recommends gnupg dirmngr \
    && gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys E81BC3078D2DD9BD \
    && gpg -a --export E81BC3078D2DD9BD | apt-key add - \
    && echo "deb http://apt.binfalse.de binfalse main" > "/etc/apt/sources.list.d/binfalse.list" \
    && echo "deb http://deb.debian.org/debian stable main" > "/etc/apt/sources.list.d/stable.list" \
    && apt-get purge -y -q --autoremove gnupg dirmngr \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# install all the dependencied
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        nagios-nrpe-server \
        monitoring-plugins \
        monitoring-plugins-basic \
        monitoring-plugins-common \
        monitoring-plugins-standard \
        nagios-plugins-contrib \
        curl \
        file \
        liburi-perl \
        liblwp-protocol-https-perl \
        libwww-perl \
        libhttp-cookies-perl \
        bf-monitoring \
        bf-syslog2stdout \
    && egrep -v "^((allowed_hosts|server_port|pid_file|include|include_dir)=|#)" /etc/nagios/nrpe.cfg | grep -v '^$' > /tmp/nrpe.cfg \
    && mv /tmp/nrpe.cfg /etc/nagios/nrpe.cfg \
    && echo "include=/etc/nagios/auto-manage.cfg" >> /etc/nagios/nrpe.cfg \
    && echo "include_dir=/etc/nagios/nrpe.d/" >> /etc/nagios/nrpe.cfg \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

VOLUME [ "/etc/nagios/nrpe.d/", "/etc/nagios/certs/", "/usr/lib/nagios/extra/" ]
EXPOSE 5666

COPY src/nrpe-runner /

ENTRYPOINT /nrpe-runner

