# Dockerfile for building container images for use in the EDK2 CI.
#
# Copyright (C) 2022, Red Hat, Inc.
# Copyright (c) 2023 Loongson Technology Corporation Limited. All rights reserved.
# Copyright (c) 2025 Damien Chen <inkfan130924783@gmai.com>. All rights reserved.
# SPDX-License-Identifier: BSD-2-Clause-Patent
#
# This file contains the definitions for images to be used for different
# jobs in the EDK2 CI pipeline. The set of tools and dependencies is split into
# multiple images to reduce the overall download size by providing images
# tailored to the task of the CI job. Currently there are two images: "build"
# and "test".
# The images are intended to run on AArch64.


# Build Image
# This image is intended for jobs that compile the source code and as a general
# purpose image. It contains the toolchains for all supported architectures, and
# all build dependencies.
#########################################
# Stage 1: build — compilation & tooling #
#########################################
FROM --platform=linux/arm64 registry.fedoraproject.org/fedora:41 AS build

# update & install core build deps in one layer, then clean caches
RUN --mount=type=cache,target=/var/cache/dnf \
    dnf install -y \
      python3 python3-virtualenv python3-devel python3-distutils-extra python3-pip \
      git make gcc gcc-c++ lcov nasm tar sudo curl \
      acpica-tools libX11-devel libXext-devel libuuid-devel \
      libasan libubsan \
      gcc-aarch64-linux-gnu gcc-arm-linux-gnu gcc-riscv64-linux-gnu gcc-loongarch64-linux-gnu \
      dotnet-runtime-9.0 nodejs npm nuget \
    && dnf clean all

# set up Python virtualenv and pip requirements
RUN python3 -m virtualenv /opt/venv \
    && /opt/venv/bin/pip install --upgrade pip setuptools \
    && /opt/venv/bin/pip install --upgrade \
         -r https://raw.githubusercontent.com/tianocore/edk2/master/pip-requirements.txt \
         lcov_cobertura

ENV VIRTUAL_ENV=/opt/venv
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

# install node-based lint tools
ARG CSPELL_VERSION=8.19.2
ARG MARKDOWNLINT_VERSION=0.44.0
RUN --mount=type=cache,target=/root/.npm \
    npm install -g cspell@${CSPELL_VERSION} markdownlint-cli@${MARKDOWNLINT_VERSION}

# toolchain prefixes
ENV GCC_AARCH64_PREFIX=/usr/bin/aarch64-linux-gnu-
ENV GCC_ARM_PREFIX=/usr/bin/arm-linux-gnu-
ENV GCC_RISCV64_PREFIX=/usr/bin/riscv64-linux-gnu-
ENV GCC_LOONGARCH64_PREFIX=/usr/bin/loongarch64-linux-gnu-
ENV GCC5_AARCH64_PREFIX=/usr/bin/aarch64-linux-gnu-
ENV GCC5_ARM_PREFIX=/usr/bin/arm-linux-gnu-
ENV GCC5_RISCV64_PREFIX=/usr/bin/riscv64-linux-gnu-
ENV GCC5_LOONGARCH64_PREFIX=/usr/bin/loongarch64-linux-gnu-

############################################
# Stage 2: test — add QEMU for CI testing  #
############################################
FROM build AS test

RUN --mount=type=cache,target=/var/cache/dnf \
    dnf install -y \
      qemu-system-arm qemu-system-aarch64 qemu-system-loongarch64 \
      qemu-system-x86 qemu-system-riscv qemu-ui-gtk \
    && dnf clean all

##############################################
# Stage 3: dev — tools for local development #
##############################################
FROM test AS dev

# install LLVM/Clang, editors and ICU
RUN --mount=type=cache,target=/var/cache/dnf \
    dnf install -y \
      clang lld llvm libicu vim nano tar curl \
    && dnf clean all

# Git Credential Manager setup
ARG GCM_VERSION=2.6.1
ARG GCM_URL="https://github.com/GitCredentialManager/git-credential-manager/releases/download/v2.0.785/gcm-osx-arm64-${GCM_VERSION}-symbols.tar.gz"

RUN curl -L "${GCM_URL}" \
      | tar -xz -C /usr/local/bin \
    && git-credential-manager-core configure \
    && git config --global credential.credentialStore cache

# copy and set our entrypoint
COPY fedora41_AArch64_dev_entrypoint.sh /usr/libexec/entrypoint
RUN chmod +x /usr/libexec/entrypoint \
    && cp /etc/skel/.bashrc /root/.bashrc

ENTRYPOINT ["/usr/libexec/entrypoint"]
