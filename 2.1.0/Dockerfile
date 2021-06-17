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

# Derecho
COPY --from=build /usr/local/include/derecho /usr/local/include/derecho
COPY --from=build /usr/local/lib/libderecho.so /usr/local/lib/libderecho.so
COPY --from=build /usr/local/lib/libderecho.so.2.1 /usr/local/lib/libderecho.so.2.1
COPY --from=build /usr/local/lib/cmake/derecho /usr/local/lib/cmake/derecho
COPY --from=build /usr/local/share/derecho /usr/local/share/derecho
COPY --from=build /usr/local/share/derecho/derecho-sample.cfg /usr/local/share/derecho/derecho-sample.cfg

# Mutils
COPY --from=build /usr/local/include/mutils /usr/local/include/mutils
COPY --from=build /usr/local/lib/libmutils.so /usr/local/lib/libmutils.so
COPY --from=build /usr/local/include/mutils-containers /usr/local/include/mutils-containers
COPY --from=build /usr/local/include/mutils-tasks /usr/local/include/mutils-tasks
COPY --from=build /usr/local/lib/libmutils-tasks.so /usr/local/lib/libmutils-tasks.so

# RDMA
COPY --from=build /usr/local/include/rdma /usr/local/include/rdma
COPY --from=build /usr/local/lib/libfabric.so.1 /usr/local/lib/libfabric.so.1
COPY --from=build /usr/lib/x86_64-linux-gnu/librdmacm.so.1 /usr/lib/x86_64-linux-gnu/librdmacm.so.1
COPY --from=build /usr/lib/x86_64-linux-gnu/libibverbs.so.1 /usr/lib/x86_64-linux-gnu/libibverbs.so.1
COPY --from=build /lib/x86_64-linux-gnu/libnl-3.so.200 /lib/x86_64-linux-gnu/libnl-3.so.200
COPY --from=build /usr/lib/x86_64-linux-gnu/libnl-route-3.so.200 /usr/lib/x86_64-linux-gnu/libnl-route-3.so.200

CMD ["/bin/bash"]