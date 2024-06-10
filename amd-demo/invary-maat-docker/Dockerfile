FROM ubuntu:24.04

COPY maat /opt/maat
COPY bin/invary /opt/invary/bin/invary
COPY bin/invary-appraiser /opt/invary-appraiser/bin/invary-appraiser
RUN mkdir -p /opt/snpguest/bin
COPY bin/snpguest /opt/snpguest/bin/snpguest
COPY bin/snpattest /opt/snpguest/bin/snpattest

RUN apt-get update && apt install -y build-essential autoconf automake libtool check    \
        libglib2.0-dev libxml2-dev libssl-dev uuid-dev libcap-dev make \
        libelf-dev libjson-c-dev libtss2-dev tpm2-tools \
        curl

RUN mkdir -p /var/opt/invary-baselines/
COPY baselines /var/opt/invary-baselines/

ENV LIBMAAT_DEBUG_LEVEL="8"
ENV LIBMAAT_LOG_SYSLOG="0"

ENTRYPOINT /opt/maat/bin/attestmgr -i 0.0.0.0:32342 -C /opt/maat/etc/maat/minimal-am-config.xml -m COPLAND

EXPOSE 32342

