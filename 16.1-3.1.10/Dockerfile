FROM postgres:16.1  AS plv8_build

LABEL org.opencontainers.image.authors="Neio Zhou <neio.zhou@gmail.com>"

ENV PLV8_VERSION=3.1.10 \
    PG_MAJOR=16


ARG BIGINT_GRACEFUL_VALUE='BIGINT_GRACEFUL=1'
ARG BIGINT_GRACEFUL
ARG BIGINT_GRACEFUL_FLAG=${BIGINT_GRACEFUL:+$BIGINT_GRACEFUL_VALUE}

RUN --mount=type=cache,target=/var/cache/apt \
    buildDependencies="build-essential \
    ca-certificates \
    curl \
    wget \
    git-core \
    python3 \
    gpp \
    cpp \
    pkg-config \
    apt-transport-https \
    cmake \
    libc++-dev \
    libc++abi-dev \
    libtinfo5 \
    libglib2.0-dev \
    ninja-build \
    postgresql-server-dev-$PG_MAJOR" \
  && runtimeDependencies="libc++1 \
    libtinfo5 \
    libc++abi1" \
  && apt-get update \
  && apt-get install -y --no-install-recommends ${buildDependencies} ${runtimeDependencies} 

RUN mkdir -p /tmp/build \
    && curl -o /tmp/build/v$PLV8_VERSION.tar.gz -SL "https://github.com/plv8/plv8/archive/v${PLV8_VERSION}.tar.gz" 

RUN cd /tmp/build \
    && tar -xzf /tmp/build/v$PLV8_VERSION.tar.gz -C /tmp/build/

RUN cd /tmp/build/plv8-$PLV8_VERSION \
    && make ${BIGINT_GRACEFUL_FLAG} static

RUN cd /tmp/build/plv8-$PLV8_VERSION \
    && make install \
    && strip /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so 


  

FROM postgres:16.1
ENV PLV8_VERSION=3.1.10 \
    PG_MAJOR=16

RUN mkdir -p /usr/lib/postgresql/${PG_MAJOR}/lib/
COPY --from=plv8_build /usr/share/postgresql/${PG_MAJOR}/extension/ /usr/share/postgresql/${PG_MAJOR}/extension/
COPY --from=plv8_build /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so /usr/lib/postgresql/${PG_MAJOR}/lib/plv8-${PLV8_VERSION}.so

RUN apt-get update \
    && apt-get install --no-install-recommends -y postgresql-${PG_MAJOR}-ip4r \
    && apt-get clean \
    && apt-get autoremove -y