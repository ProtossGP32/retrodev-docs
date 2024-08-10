# 1st stage: prepare the base image for builds
FROM debian:bookworm-slim AS base
# Install dependencies
RUN apt update && apt install --no-install-recommends -y \
    # Common build dependencies
    cmake make g++ gcc \
    # phosg build dependencies
    zlib1g-dev python3 \
    # newserv build dependencies
    libevent-dev \
    && rm -rf /var/lib/apt/lists/*

# Build target
FROM base as build
# Define some argument variables
ARG BUILD_TYPE=Release
# First build and install phosg
COPY phosg /tmp/build/phosg
WORKDIR /tmp/build/phosg
RUN cmake . && make && make test && make install
# After that, build and install resource_dasm
COPY resource_dasm /tmp/build/resource_dasm
WORKDIR /tmp/build/resource_dasm
RUN cmake . && make && make install
# Then build newserv
COPY newserv /tmp/build/newserv
WORKDIR /tmp/build/newserv
# - Configure CMake and build
RUN cmake . && make

# Release target
FROM debian:bookworm-slim as release
# Install runtime dependencies
RUN apt update && apt install --no-install-recommends -y \
    # phosg runtime dependencies
    zlib1g \
    # newserv runtime dependencies
    libevent-dev \
    && rm -rf /var/lib/apt/lists/*
# Copy required libraries and objects
#COPY --from=build /usr/local/lib/libphosg.a /usr/local/lib/
#COPY --from=build /usr/local/lib/libresource_file.a /usr/local/lib/
#COPY --from=build /usr/local/include/resource_file /usr/local/include/resource_file
#COPY --from=build /usr/local/include/resource_file /usr/local/include/resource_file
# Set the workdir for newserv
WORKDIR /usr/newserv
# Copy all binaries
COPY --from=build /usr/local/bin /usr/local/bin/
COPY --from=build /tmp/build/newserv/newserv /usr/local/bin/newserv
# Copy newserv files
COPY --from=build /tmp/build/newserv/system /usr/newserv/system
# Set a default config file within the image
#RUN cp system/config.example.json system/config.json
ENTRYPOINT [ "newserv" ]