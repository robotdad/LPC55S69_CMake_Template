# See here for image contents: https://github.com/microsoft/vscode-dev-containers/tree/v0.195.0/containers/cpp/.devcontainer/base.Dockerfile
# [Choice] Debian / Ubuntu version (use Debian 11/9, Ubuntu 18.04/21.04 on local arm64/Apple Silicon): debian-11, debian-10, debian-9, ubuntu-21.04, ubuntu-20.04, ubuntu-18.04
ARG VARIANT=debian-11
FROM mcr.microsoft.com/vscode/devcontainers/base:0-${VARIANT} AS dev

# [Optional] Uncomment this section to install additional packages.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install libncurses5 --no-install-recommends 

# get vcpkg
ADD https://aka.ms/vcpkg-init.sh /tmp
RUN /bin/bash -c "chmod +x /tmp/vcpkg-init.sh && . /tmp/vcpkg-init.sh"
WORKDIR /src
COPY ./vcpkg-configuration.json /src/vcpkg-configuration.json
RUN /bin/bash -c ". ~/.vcpkg/vcpkg-init && vcpkg activate"

# non interactive build env
FROM dev as build
COPY ./ /src
RUN /bin/bash -c ". ~/.vcpkg/vcpkg-init && vcpkg activate && cmake --preset m33-debug && cmake --build --preset m33-debug"
RUN /bin/bash -c ". ~/.vcpkg/vcpkg-init && vcpkg activate && cmake --preset m33-release && cmake --build --preset m33-release"



