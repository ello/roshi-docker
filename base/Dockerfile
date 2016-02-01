FROM ubuntu-debootstrap:latest

# Add roshi user and group
RUN groupadd --system roshi && \
    useradd --system --gid roshi --shell /bin/false roshi

# Install tools
RUN apt-get update && \
    apt-get install -y curl git make && \
    rm -rf /var/lib/apt/lists/*

# Install gosu
# https://github.com/tianon/gosu
RUN gpg --keyserver pool.sks-keyservers.net --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4
RUN curl -o /usr/local/bin/gosu -SL "https://github.com/tianon/gosu/releases/download/1.4/gosu-$(dpkg --print-architecture)" && \
    curl -o /usr/local/bin/gosu.asc -SL "https://github.com/tianon/gosu/releases/download/1.4/gosu-$(dpkg --print-architecture).asc" && \
    gpg --verify /usr/local/bin/gosu.asc && \
    rm /usr/local/bin/gosu.asc && \
    chmod +x /usr/local/bin/gosu

# Install Godeb
# https://github.com/niemeyer/godeb
RUN curl -Ss -o /tmp/godeb-amd64.tar.gz https://godeb.s3.amazonaws.com/godeb-amd64.tar.gz && \
    tar -xvzf /tmp/godeb-amd64.tar.gz -C /usr/local/bin && \
    rm /tmp/godeb-amd64.tar.gz

# Envs
ENV ROSHI_COMMIT="53e6bd85a0fe12befe5d0fd0888070f671cb7ddd"
ENV GO_VERSION="1.4.2"

# Install Go, build roshi-server and roshi-walker, then delete Go
RUN godeb install ${GO_VERSION} && \
    export GOPATH=/go && export PATH=$PATH:${GOPATH}/bins && \
    mkdir -p ${GOPATH}/src/github.com/soundcloud && \
    git clone https://github.com/soundcloud/roshi.git ${GOPATH}/src/github.com/soundcloud/roshi && \
    cd ${GOPATH}/src/github.com/soundcloud/roshi && git checkout ${ROSHI_COMMIT} && \
    cd roshi-server && make && mv roshi-server /usr/local/bin && \
    cd ../roshi-walker && make && mv roshi-walker /usr/local/bin && \
    cd / && \
    rm -rf ${GOPATH} && \
    rm -rf /usr/local/go && \
    unset GOPATH

# Entrypoint
EXPOSE 6302
ENTRYPOINT ["gosu", "roshi:roshi", "roshi-server"]
