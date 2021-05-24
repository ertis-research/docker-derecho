FROM ubuntu:20.04 as build

# Avoid timezone issue
ARG DEBIAN_FRONTEND=noninteractive

# Install avaiable dependencies from ubuntu repositories
RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        wget \
        git \
        build-essential \
        cmake \
        libtool \
        m4 \
        automake \
        libssl-dev \
        librdmacm-dev \
        libibverbs-dev

# Download Derecho source
RUN git clone -b v2.1.0 --depth 1 https://github.com/Derecho-Project/derecho.git

# Install libfabric
RUN ./derecho/scripts/prerequisites/install-libfabric.sh

# Instal mutils
RUN ./derecho/scripts/prerequisites/install-mutils.sh

# Instal mutils containers
RUN ./derecho/scripts/prerequisites/install-mutils-containers.sh

# Instal mutils tasks
RUN ./derecho/scripts/prerequisites/install-mutils-tasks.sh

# Instal libspdlog 1.3 (1.5 will not work with Derecho)
RUN wget http://old-releases.ubuntu.com/ubuntu/pool/universe/s/spdlog/libspdlog-dev_1.3.1-1_amd64.deb \
    && dpkg -i libspdlog-dev_1.3.1-1_amd64.deb

# Build and install Derecho
WORKDIR /derecho/Release
RUN cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local .. \
    && make -j $(nproc) \
    && make install

# Copy installed files to second ubuntu stage to save space
FROM ubuntu:20.04

COPY --from=build /usr/local/include/derecho /usr/local/include/derecho
COPY --from=build /usr/local/lib/libderecho.so /usr/local/lib/libderecho.so
COPY --from=build /usr/local/lib/cmake/derecho /usr/local/lib/cmake/derecho
COPY --from=build /usr/local/share/derecho /usr/local/lib/share/derecho
COPY --from=build /usr/local/share/derecho/derecho-sample.cfg /usr/local/share/derecho/derecho-sample.cfg

CMD ["/bin/bash"]