FROM maven:3.6.3-openjdk-11

# Copy maven settings, containing repository configurations
RUN mkdir -p /var/jenkins_home/agent/.m2
COPY settings.xml /var/jenkins_home/agent/.m2
COPY settings.xml /root/.m2

# Install Docker
RUN apt-get update -y
RUN apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
RUN apt-key fingerprint 0EBFCD88
RUN add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
RUN apt-get update -y
RUN apt-get install -y \
    docker-ce \
    docker-ce-cli \
    containerd.io

# Install Docker Compose
RUN curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
RUN chmod +x /usr/local/bin/docker-compose

ENV GOSU_VERSION=1.12 \
    SWARM_VERSION=3.23 \
    MD5=5507abb36d6ca8b01f3416ef21758c9d \
    OPENSSL_CONF=/etc/ssl/

# grab gosu for easy step-down from root
RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates wget bzip2 python \
 && rm -rf /var/lib/apt/lists/* \
 && wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture)" \
 && wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$(dpkg --print-architecture).asc" \
 && export GNUPGHOME="$(mktemp -d)" \
 && GPG_KEYS=B42F6819007F00F88E364FD4036A9C25BF357DD4 \
 && gpg --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$GPG_KEYS" \
  || gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEYS" \
  || gpg --keyserver pgp.mit.edu --recv-keys "$GPG_KEYS" \
  || gpg --keyserver keyserver.pgp.com --recv-keys "$GPG_KEYS" \
 && gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu \
 && rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc \
 && chmod +x /usr/local/bin/gosu \
 && gosu nobody true

# grab swarm-client.jar
RUN mkdir -p /var/jenkins_home \
 && useradd -d /var/jenkins_home/agent -u 1000 -m -s /bin/bash jenkins \
 && curl -o /bin/swarm-client.jar -SL https://repo.jenkins-ci.org/releases/org/jenkins-ci/plugins/swarm-client/$SWARM_VERSION/swarm-client-$SWARM_VERSION.jar \
 && echo "$MD5  /bin/swarm-client.jar" | md5sum -c -

COPY docker-entrypoint.sh /
RUN chmod +x docker-entrypoint.sh

# Switch from `sh -c` to `bash -c` as the shell behind a `RUN` command.
SHELL ["/bin/bash", "-c"]

# Usual updates
RUN apt-get update && apt-get upgrade -y
# Dependencies for sdkman installation
RUN apt-get install curl bash unzip zip -y

#Install sdkman
RUN curl -s "https://get.sdkman.io" | bash

# FUN FACTS:
# 1) the `sdk` command is not a binary but a bash script loaded into memory
# 2) Shell sessions are a "process", which means environment variables
#    and declared shell function only exist for 
#    the duration that shell session exists
RUN source "$HOME/.sdkman/bin/sdkman-init.sh" \
    && sdk install sbt 1.4.2 \
    && sdk install scala 2.12.12

# Once the real binaries exist these are 
# the symlinked paths that need to exist on PATH
ENV PATH=/root/.sdkman/candidates/scala/current/bin:$PATH
ENV PATH=/root/.sdkman/candidates/sbt/current/bin:$PATH

VOLUME /var/jenkins_home/agent
WORKDIR /var/jenkins_home/agent

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["java", "-jar", "/bin/swarm-client.jar"]