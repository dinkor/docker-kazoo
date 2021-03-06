FROM debian:jessie-slim
MAINTAINER Sergey Korobkov <skorobkov@dinkor.net>

ARG KAZOO_REPO=https://github.com/2600hz/kazoo.git
ARG KAZOO_BRANCH=master
ARG USER=kazoo
ARG DUMB_INIT_VERSION=1.2.0
ARG ERLANG_VERSION=18.3
ARG KAZOO_DIR=/opt/kazoo

ENV PATH=/tmp/erlang/bin:${PATH}

COPY kazoo_* /usr/local/sbin/

RUN echo 'APT::Install-Recommends "false";' > /etc/apt/apt.conf.d/build_docker_norecommends \
    && echo 'APT::Get::AutomaticRemove "true";' >> /etc/apt/apt.conf.d/build_docker_norecommends \
    && echo 'APT::AutoRemove::RecommendsImportant "false";' >> /etc/apt/apt.conf.d/build_docker_norecommends \
    && echo 'APT::AutoRemove::SuggestsImportant "false";' >> /etc/apt/apt.conf.d/build_docker_norecommends \
    && apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y \
	    build-essential \
	    curl \
	    expat \
	    git-core \
	    libexpat1-dev \
	    libncurses5-dev \
	    libssl-dev \
	    libxslt-dev \
	    python \
	    wget \
	    zlib1g-dev \
	    ca-certificates \
	    htmldoc \
	    unzip \
	    zip \
    && curl -fSL -o /tmp/kerl https://raw.githubusercontent.com/kerl/kerl/master/kerl \
    && chmod +x /tmp/kerl \
    && /tmp/kerl update releases \
    && /tmp/kerl build ${ERLANG_VERSION} ${ERLANG_VERSION} \
    && /tmp/kerl install ${ERLANG_VERSION} /tmp/erlang \
    && /tmp/kerl cleanup all \

    && git clone --depth 1 --single-branch -b ${KAZOO_BRANCH} ${KAZOO_REPO} /tmp/kazoo_build \
    && make -C /tmp/kazoo_build deps compile build-release \

    && useradd --user-group --create-home --home-dir ${KAZOO_DIR} --shell /bin/false ${USER} \
    && mv /tmp/kazoo_build/_rel/kazoo/* ${KAZOO_DIR} \

    && curl -fSL -o /tmp/dumb-init_${DUMB_INIT_VERSION}_amd64.deb https://github.com/Yelp/dumb-init/releases/download/v${DUMB_INIT_VERSION}/dumb-init_${DUMB_INIT_VERSION}_amd64.deb \
    && dpkg -i /tmp/dumb-init_*_amd64.deb \
    && chown -R ${USER}:${USER} ${KAZOO_DIR} \
    && chmod +x /usr/local/sbin/kazoo_* \

    && apt-get purge -y \
	    build-essential \
	    curl \
	    expat \
	    git-core \
	    libexpat1-dev \
	    libncurses5-dev \
	    libssl-dev \
	    libxslt-dev \
	    python \
	    wget \
	    zlib1g-dev \

    && apt-get clean \
    && rm -f /etc/apt/apt.conf.d/build_docker_norecommends \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf ~/.kerl/ \
    && rm -rf /tmp/*


USER ${USER}
WORKDIR ${KAZOO_DIR}
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["kazoo_start"]
