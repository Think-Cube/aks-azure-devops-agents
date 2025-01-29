FROM ubuntu:22.04

ENV TARGETARCH="linux-x64"

RUN apt update && \
    apt upgrade -y && \
    apt install -y curl git jq libicu70 docker.io unzip zip

WORKDIR /azp/

COPY ./start.sh ./
RUN chmod +x ./start.sh

RUN useradd -m -d /home/agent agent && \
    chown -R agent:agent /azp /home/agent

USER agent

ENTRYPOINT ["bash", "./start.sh"]
