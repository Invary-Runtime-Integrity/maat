FROM MY_REGISTRY/invary-demo/invary-maat:latest

ENV LIBMAAT_DEBUG_LEVEL="3"
ENV LIBMAAT_LOG_SYSLOG="0"

COPY motd /etc/motd
COPY bashrc /etc/bash.bashrc
RUN mkdir /invary
WORKDIR /invary
COPY appraise /invary/appraise
RUN chmod +x /invary/appraise
COPY enable-confidentiality.sh /invary/enable-confidentiality.sh
COPY disable-confidentiality.sh /invary/disable-confidentiality.sh
RUN chmod +x /invary/enable-confidentiality.sh
RUN chmod +x /invary/disable-confidentiality.sh

RUN apt-get update && apt install -y curl python3 python3-venv pip vim

RUN python3 -m venv /invary/venv
RUN . /invary/venv/bin/activate
RUN /invary/venv/bin/pip install kubernetes-client

ENTRYPOINT sleep 2392000

